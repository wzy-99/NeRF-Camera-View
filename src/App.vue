<script setup>
import Plotly from 'plotly.js-dist'
import { onMounted } from 'vue'

import { ref } from 'vue'

const currentImage = ref(null)

const name2url = {}

var viewer = null

const layout = {
	"scene": {
		"camera": {
			"eye": { x: -0.76, y: 1.8, z: 0.92 }
		}
	},
	width: 800,
	height: 600,
}

const coneSize = 0.2;

onMounted(() => {
	const data = [
		{
			x: [],
			y: [],
			z: [],
			u: [],
			v: [],
			w: [],
			type: "cone",
			showscale: false
		},
	]
	Plotly.newPlot('viewer', data, layout).then((g) => {
		viewer = g
		viewer.on('plotly_click', onConeClick)
		viewer.on('plotly_hover', onConeClick)
	})
})

const onJSONUpload = (e) => {
	const file = e.target.files[0]
	readJSON(file).then((json) => {
		drawCone(json)
	})
}

const readJSON = (file) => {
	const reader = new FileReader()
	reader.readAsText(file)
	return new Promise((resolve, reject) => {
		reader.onload = () => {
			const json = JSON.parse(reader.result)
			resolve(json)
		}
	})
}

const drawCone = (json) => {
	const frames = json.frames

	const data = []

	for (let i = 0; i < frames.length; i++) {
		const frame = frames[i]
		const transform_matrix = frame.transform_matrix
		const file_path = frame.file_path.split("/").pop()
		const x = transform_matrix[0][3]
		const y = transform_matrix[1][3]
		const z = transform_matrix[2][3]
		const v = (transform_matrix[0][0] + transform_matrix[1][0] + transform_matrix[2][0])
		const u = (transform_matrix[0][1] + transform_matrix[1][1] + transform_matrix[2][1])
		const w = -(transform_matrix[0][2] + transform_matrix[1][2] + transform_matrix[2][2]) 
		data.push({
			x: [x],
			y: [y],
			z: [z],
			u: [u],
			v: [v],
			w: [w],
			type: "cone",
			showscale: false,
			sizemode: "absolute",
			sizeref: coneSize,
			hoverinfo: file_path,
			name: file_path
		})
	}

	// set data
	viewer.data = data

	Plotly.redraw(viewer)
}

const onImageUpload = (e) => {
	for (const file of e.target.files) {
		const reader = new FileReader()
		reader.readAsDataURL(file)
		reader.onload = () => {
			const name = file.name
			const url = reader.result
			name2url[name] = url
		}
	}
}

const onConeClick = (data) => {
	const point = data.points[0]
	const name = point.data.name
	if (name) {
		if (name2url[name]) {
			currentImage.value = name2url[name]
		}
	}
}

</script>

<template>
	<div class="hero min-h-screen bg-base-100">
		<div class="hero-content text-center">
			<div id="viewer" class="border-2 border-base-300 rounded-box"></div>
			<div class="flex flex-col gap-1">
				<h1 class="text-4xl font-bold">3D Viewer</h1>
				<img id="image" :src="currentImage || '/image-fill.png'" width="400" height="400" class="rounded-box border-2 border-base-300">
				<label for="json" class="btn">Select JSON</label>
				<input id="json" type="file" class="hidden" @change="onJSONUpload" accept=".json">
				<label for="images" class="btn">Select IMAGE</label>
				<input id="images" type="file" class="hidden" @change="onImageUpload" accept="image/*" multiple>
			</div>
		</div>
	</div>
</template>