<template>
	<div class="container">
		<div>
			<h2>Audio File Preview</h2>
			<hr/>
			<label>Audio File
				<input type="file" accept="audio/*" @change="handleFileUpload( $event )"/>
			</label>
			<br>
			<br>
			<audio id="audio-preview" controls v-show="file != ''"/>
			<br>
			<button v-on:click="submitFile()">Submit File</button>
			<br>
			<br>
			<br>
			<br>
			<br>
			<br>
			<br>
		</div>
	</div>
	<div class="container">
		<div>
			<div class="w3-container"> 
			<h2>Transcript of File</h2>
			<p> {{ fileTranscript }} </p>
			<br>
			<br>
			<br>
			<br>
			</div>
		</div>
	</div>

</template>

<script>
	import axios from 'axios';
	
	export default {

		name: "AudioTranscriber",
		props: {
			msg: String
		},
		
		data(){
			return {
			file: null, 
			fileUuid: '',
			fileTranscript: ''
			}
		},
		
		methods: {
			handleFileUpload( event ){
				this.file = event.target.files[0];
				this.previewAudio();
			},
			// preview the audio file
			previewAudio(){
				let audio = document.getElementById('audio-preview');
				let reader = new FileReader();
				reader.readAsDataURL( this.file );
				reader.addEventListener('load', function(){
					audio.src = reader.result;
				});
			},
			async submitFile(){
				// use formdata to submit the file
				let formData = new FormData();				
				formData.append('file', this.file);
				


				// Upload file to cloud function
				let fileUploadResponse = await axios.put( 'https://us-central1-neon-vigil-331620.cloudfunctions.net/upload_audio_file_to_gcs',
					formData,
					{
						headers: {
							"Access-Control-Allow-Origin": `http://localhost:8080`,
							'Content-Type': 'multipart/form-data'
						}
					}
				);

				// Get uuid from response
				this.fileUuid = fileUploadResponse.data.uuid;
				console.log(fileUploadResponse);

				let getConfig = {
					params: {
						uuid: this.fileUuid,
						isLoop: "true"
					}
				}

				// Call function that will get the transcript once it appears in Firestore then display it on the website
				let getTranscriptResponse = await axios.get(`https://us-central1-neon-vigil-331620.cloudfunctions.net/get_transcript_by_file_uuid`, getConfig)
				
				if (getTranscriptResponse.data.data && typeof(getTranscriptResponse.data.data) === 'string') {
					this.fileTranscript = getTranscriptResponse.data.data;
				} else {
					// if error, display this error message
					console.log(getTranscriptResponse);
					this.fileTranscript = "There was an error getting the transcript.";
				}


			}
		}
	}
</script>