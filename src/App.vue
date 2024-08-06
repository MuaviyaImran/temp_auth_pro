<script setup>
const signInWithGoogle = () => {
  const client_id = import.meta.env.VITE_GOOGLE_CLIENT_ID;
  gapi.load("client:auth2", async function () {
    const auth = await gapi.auth2.init({
      client_id: client_id,
      scope: "https://www.googleapis.com/auth/cloud-platform",
    });
    await auth.signIn().then(async (user) => {
      const userData = {
        google_id_token: user.Sc.id_token,
        profile_image: user.my.xU,
        name: user.my.dg,
      };
      console.log("Access Token", userData.google_id_token);
      console.log(userData);
      try {
        const response = await fetch("/auth/login", {
          method: "POST",
          headers: {
            "Content-Type": "application/json",
          },
          body: JSON.stringify(userData),
        });
        const userDetails = await response.json();
        const redirectURL = parseRedirectURL(window.location.href);
        userDetails.jwt = response.headers.get("Jwt-Token");
        userDetails.Refresh_Token = response.headers.get("Refresh-Token");
        userDetails.redirectURL = redirectURL;
        if (userDetails.jwt && userDetails.redirectURL) {
          window.location = `${userDetails.redirectURL}?jwt=${userDetails.jwt}&refresh_token=${userDetails.Refresh_Token}`;
        } else console.log("Success:", userDetails);
      } catch (error) {
        console.error("Error:", error);
      }
    });
  });
};
function parseRedirectURL(url) {
  var queryString = url.split("?")[1];
  var queryParams = new URLSearchParams(queryString);
  return queryParams.get("redirect_url");
}
var redirectURL = parseRedirectURL(window.location.href);
</script>

<template>
  <section class="main" hx-swap-oob="true">
    <p class="auth-text">Sign in with your Datum Brain ID</p>
    <button class="login-button" @click="signInWithGoogle">
      <span> Sign in with Google </span>
    </button>
    <div id="data"></div>
  </section>
</template>

<style scoped>
.logo {
  height: 6em;
  padding: 1.5em;
  will-change: filter;
  transition: filter 300ms;
}
.logo:hover {
  filter: drop-shadow(0 0 2em #646cffaa);
}
.logo.vue:hover {
  filter: drop-shadow(0 0 2em #42b883aa);
}
</style>
