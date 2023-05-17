<template>
	<div>
		<input type="file" @change="selectFile" />
		<button @click="upload">上传</button>
		<div v-for="(chunkProgress, index) in chunkProgresses" :key="index">
			<div>{{ index + 1 }}号分片进度：{{ chunkProgress }}%</div>
			<div class="progress-bar">
				<div class="progress" :style="{ width: chunkProgress + '%' }"></div>
			</div>
		</div>
	</div>
</template>

<script>
	export default {
		data() {
			return {
				file: null,
				progress:0,//总进度
				chunkProgresses: [], // 存储每个分片的进度信息
			};
		},
		methods: {
			selectFile(event) {
				this.file = event.target.files[0];
			},
			upload() {
				const chunkSize = 1024 * 1024; // 每个分片大小（1MB）
				const totalChunks = Math.ceil(this.file.size / chunkSize);
				let currentChunk = 0;

				const uploadChunk = () => {
					const start = currentChunk * chunkSize;
					const end = Math.min(start + chunkSize, this.file.size);
					const chunk = this.file.slice(start, end);

					const chunkProgress = {
						index: currentChunk + 1,
						progress: 0,
					};

					this.chunkProgresses.push(chunkProgress); // 添加当前分片的进度信息

					uni.uploadFile({
						url: 'http://your-server-url.com/upload',
						filePath: chunk.path,
						name: 'file',
						success: (res) => {
							currentChunk++;
							chunkProgress.progress = 100; // 设置当前分片的进度为100%
							this.progress = Math.floor((currentChunk / totalChunks) * 100);
							if (currentChunk < totalChunks) {
								uploadChunk();
							} else {
								// 所有分片上传完成
								// 在这里进行后续操作，如合并分片等
							}
						},
						fail: (err) => {
							console.error(err);
						},
						onProgressUpdate: (res) => {
							// 更新当前分片的进度信息
							chunkProgress.progress = Math.floor((res.totalBytesSent / res
								.totalBytesExpectedToSend) * 100);
						},
					});
				};

				uploadChunk();
			},
		},
	};
</script>

<style>
	.progress-bar {
		width: 100%;
		height: 20px;
		background-color: #ccc;
	}

	.progress {
		height: 100%;
		background-color: #42b983;
		transition: width 0.3s ease-in-out;
	}
</style>

