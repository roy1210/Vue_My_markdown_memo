<template>
  <div id="home">
    <h1>
      <img alt="MyMarkdown" src="../assets/logo.png">
      {{msg}}
    </h1>
    <button @click="googleLogin">Login with Google account</button>
  </div>
</template>
<script>
export default {
  name: "home",
  data() {
    return {
      msg: "Welcome to My Markdown Memo"
    };
  },
  methods: {
    googleLogin: function() {
      firebase
        .auth()
        .signInWithRedirect(new firebase.auth.GoogleAuthProvider());
    }
  }
};
</script>

<style lang="scss">
.home {
  margin-top: 60px;
}
</style>
