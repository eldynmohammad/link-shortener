<template>
	<div class="container grid">
		<div class="box-left illustration">
			<div class="section">
				<h3 class="illustration__title">Link Shortener</h3>
				<img class="illustration__img" src="/img/link.svg" alt="">
			</div>
		</div>
		<div class="box-right">
			<div class="section input">
				<input type="text" v-model="link" class="input__input" placeholder="https://www.example.co">
				<button class="input__button" @click="getNewUrl">Generate</button>
			</div>
			<div class="section output">
				<p class="output__text">Link shorten:</p>
				<div class="output__link">
					<span class="output__link-link" id="new-link">{{ newLink }}</span>
					<button class="output__link-button"  data-clipboard-target="#new-link" v-if="exist" @click="showAlert">Copy</button>
				</div> 
			</div>
		</div>
	</div>
	<div class="alert">
		Link copied!
	</div>
</template>

<script>
import ClipboardJS from 'clipboard';
export default {
	data() {
		return {
			link: '',
            newLink: '',
            token: '1a7dcc341b36ae24539b4abd3a2f8fd55e165b52',
            groupID: '',
            errorMessage: '',
            exist: false,
        }
    },
    methods: {
        getNewUrl() {
			if(this.link.length === 0) {
               return;
            }
            const options = {
				method: "POST",
                headers: { 
                    "Authorization": `Bearer ${this.token}`,
                    "Content-Type": "application/json" 
                },
                body: JSON.stringify({ "long_url": this.link, "domain": "bit.ly", "group_guid": this.groupID })
            }
            fetch('https://api-ssl.bitly.com/v4/shorten', options)
            .then(response => response.json())
            .then(data => {
                if(data.message === "INVALID_ARG_LONG_URL") {
					this.errorMessage = "Invalid URL to shorten !";
                    return;
                }
                
                const url = data;
                this.newLink = url.link;
                this.exist = true;
                this.link = '';
            })
            .catch(error => {
				console.error('There was an error!', error);
            });
        },
		showAlert() {
			const alert_popup = document.querySelector('.alert') 
			alert_popup.classList.add("show-alert")

			setTimeout(
				() => {
					alert_popup.classList.remove("show-alert")
				}, 6000
			)
		}
    },
	created() {
		fetch('https://api-ssl.bitly.com/v4/groups', {
			method: "GET",
			headers: {
				"Authorization": `Bearer ${this.token}`,
				"Content-Type": "application/json" 
			}
		})
		.then(response => response.json())
		.then(data => this.groupID = data.groups[0].guid);
		new ClipboardJS('.output__link-button');
	},
}
</script>

