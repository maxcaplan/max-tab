<template>
  <div id="background">
    <div id="content" class="p-3">
      <weather />
    </div>
  </div>
</template>

<script>
import Weather from "../components/Weather.vue";

export default {
  name: "App",

  components: {
    Weather,
  },
};
</script>

<style>
html,
body {
  width: 100%;
  height: 100%;
  margin: 0;
  border: 0;
}

#background {
  background: linear-gradient(227deg, #7f00ff, #e100ff);
  background-size: 400% 400%;

  width: 100%;
  height: 100%;

  -webkit-animation: bg-scroll 30s ease infinite;
  -moz-animation: bg-scroll 30s ease infinite;
  animation: bg-scroll 30s ease infinite;
}

@-webkit-keyframes bg-scroll {
  0% {
    background-position: 0% 50%;
  }
  50% {
    background-position: 100% 50%;
  }
  100% {
    background-position: 0% 50%;
  }
}
@-moz-keyframes bg-scroll {
  0% {
    background-position: 0% 50%;
  }
  50% {
    background-position: 100% 50%;
  }
  100% {
    background-position: 0% 50%;
  }
}
@keyframes bg-scroll {
  0% {
    background-position: 0% 50%;
  }
  50% {
    background-position: 100% 50%;
  }
  100% {
    background-position: 0% 50%;
  }
}
</style>
