<template>
  <div v-if="icon">
    <!-- Custom svg icon -->
    <object
      v-if="icoName == 'few-clouds' || icoName == 'mist'"
      width="56"
      height="56"
      fill="none"
      stroke="currentColor"
      stroke-width="2"
      stroke-linecap="round"
      stroke-linejoin="round"
      type="image/svg+xml"
      v-bind="{ data: require('../assets/icons/' + iconMap[icon] + '.svg') }"
    ></object>

    <!-- Feather scg icon -->
    <svg
      v-else
      width="56"
      height="56"
      fill="none"
      stroke="currentColor"
      stroke-width="2"
      stroke-linecap="round"
      stroke-linejoin="round"
    >
      <use
        v-bind="{
          'xlink:href':
            require('../assets/icons/feather-sprite.svg') + '#' + icoName,
        }"
      />
    </svg>
  </div>
</template>

<script>
export default {
  name: "WeatherIcons",

  props: { icon: String },

  data() {
    return {
      iconMap: {
        "01d": "sun",
        "02d": "few-clouds",
        "03d": "cloud",
        "04d": "cloud",
        "09d": "cloud-drizzle",
        "10d": "cloud-rain",
        "11d": "cloud-lightning",
        "13d": "cloud-snow",
        "50d": "mist",
      },
    };
  },

  computed: {
    // Convert OpenWeather icon codes to feather-icon names
    icoName() {
      let icon = this.icon;
      if (icon.charAt(icon.length - 1) == "n")
        icon = icon.slice(0, icon.length - 1) + "d";

      return this.iconMap[icon];
    },
  },
};
</script>

<style></style>
