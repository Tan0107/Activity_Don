<template>
    <h1 class="text-center my-4 pt-5" id="contact">Contact</h1>
        <div class="contact-section">
            <div class="row align-items-center mt-4">
                <div class="col-md-6 map-container">
                    <iframe id="gmap_canvas" src="https://maps.google.com/maps?q=centro%20escolar%20university%20manila&t=&z=13&ie=UTF8&iwloc=&output=embed" frameborder="0" scrolling="no" marginheight="0" marginwidth="0"></iframe>
                </div>
                <div class="col-md-6">
                    <form @submit.prevent="submitForm">
                        <div class="mb-3">
                            <input type="text" v-model="name" class="form-control contact-form-control" placeholder="First Name M.I. Last Name" required>
                        </div>
                        <div class="mb-3">
                            <input type="email" v-model="email" class="form-control contact-form-control" placeholder="Email" required>
                        </div>
                        <div class="mb-3">
                            <textarea v-model="message" class="form-control contact-form-control" rows="6" placeholder="Message" required></textarea>
                        </div>
                        <div class="form-footer">
                            <div class="social-icons">
<!--                                <a href="https://www.facebook.com/profile.php?id=100085701498879" id="facebook"><i class="fab fa-facebook"></i></a> -->
                                <a href="https://www.linkedin.com/in/charles-babbage-8291a6211/" id="linkedin"><i class="fab fa-linkedin"></i></a>
                                <a href="https://gitlab.com/cbabbage0991" id="gitlab"><i class="fab fa-gitlab"></i></a>
                                <a href="https://github.com/cbabbage0991" id="github"><i class="fab fa-github"></i></a>
                            </div>
                            <button type="submit" class="submit-btn pl-5 pr-5" :disabled="isLoading">{{isLoading ? "Sending..." : "Submit"}}</button>
                        </div>

                        <div class="d-flex justify-content-end mt-2">
                            <div ref="recaptchaContainer"></div>
                        </div>
                    </form>
                    
                </div>
            </div>
        </div>
        
</template>

<script setup>

    import { ref, onMounted, onBeforeUnmount } from 'vue';
    import { Notyf } from 'notyf';
    import 'notyf/notyf.min.css';

    const notyf = new Notyf();
    const WEB3FORMS_ACCESS_KEY = "7b2d9ab7-d2ba-4ebb-a035-7a0ea66439fa"; //replace with your access key

    const subject = "New message from portfolio contact form";

    const name = ref("");
    const email = ref("");
    const message = ref("");

    const isLoading = ref(false);

    // The submitForm() handler function sends the form data to web3forms and displays success or failure notifications toast.
    const submitForm = async () => {

        if(!recaptchaToken.value){
            notyf.error('Please verify that you are not a robot.');
            return;
        }

        // Set the loading state to true.
        isLoading.value = true;
        try{
            // Send the form data to web3forms using fetch() API.
            // The fetch() API is used to send HTTP requests to a server.
            // Sends a POST request to the https://api.web3forms.com/submit endpoint.
            // The request body contains the form data and the web3forms access key.
            const response = await fetch("https://api.web3forms.com/submit", {
                method: "POST",
                headers: {
                    "Content-Type": "application/json",
                    Accept: "application/json",
                },
                body: JSON.stringify({
                    access_key: WEB3FORMS_ACCESS_KEY,
                    subject: subject,
                    name: name.value,
                    email: email.value,
                    message: message.value,
                }),
            });
            const result = await response.json();

            // Check if the form submission was successful.
            // If successful, display success notification toast.
            if (result.success) {
                console.log(result);
                // Set the loading state to false.
                isLoading.value = false;
                notyf.success("Message Sent!");
            }
        } catch (error) {

            // If not successful, display failure notification toast.
            console.log(error);
            // Set the loading state to false.
            isLoading.value = false;
            notyf.error("Failed to send message.");
        } finally {
            resetRecaptcha();
        }
    }

    /** 
 * 
 * reCaptcha Integration 
 * 
 */

const SITE_KEY = '6LdmvpIrAAAAAIg-hwItyWQzsFbJ_FMTYZQuT_-8';  // Replace with your site key

const recaptchaContainer = ref(null);
const recaptchaWidgetId = ref(null);
const recaptchaToken = ref('');

// Callback called by reCAPTCHA when successful
function onRecaptchaSuccess(token) {
  recaptchaToken.value = token;
}

// Callback when expired
function onRecaptchaExpired() {
  recaptchaToken.value = '';
}

// Function to render the reCAPTCHA widget
function renderRecaptcha() {
  if (!window.grecaptcha) {
    console.error('reCAPTCHA not loaded');
    return;
  }

  recaptchaWidgetId.value = window.grecaptcha.render(recaptchaContainer.value, {
    sitekey: SITE_KEY,
    size: 'normal', // or 'compact'
    callback: onRecaptchaSuccess,
    'expired-callback': onRecaptchaExpired,
  });
}

// Function to reset reCAPTCHA 
function resetRecaptcha() {
  if (recaptchaWidgetId.value !== null) {
    window.grecaptcha.reset(recaptchaWidgetId.value);
    recaptchaToken.value = '';
  }
}



onMounted(() => {
  // This code waits for the Google reCAPTCHA library to load, then renders the reCAPTCHA widget using onMounted hook. 
  // The widget is rendered with grecaptcha.render(), which requires a sitekey. 
  // Callback functions handle success and expiration events. 
  // reCAPTCHA is reset upon form submission to clear the token.
  const interval = setInterval(() => {
    if (window.grecaptcha && window.grecaptcha.render) {
      renderRecaptcha();
      clearInterval(interval);
    }
  }, 100);

  onBeforeUnmount(() => {
    clearInterval(interval);
  });
});
    
</script>