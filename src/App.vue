<!--
SPDX-FileCopyrightText: NOI Techpark <digital@noi.bz.it>

SPDX-License-Identifier: AGPL-3.0-or-later
-->

<template>
  <EventsNOI
    :options="{
      room: room,
      fontName: fontName,
    }"
  />
</template>
<script>
import EventsNOI from "./components/EventsNOI.vue";
export default {
  name: "App",
  props: {
    room: { type: String, default: "" },
    fontName: {
      type: String,
      default: "Source Sans Prod",
    },
  },
  components: {
    EventsNOI,
  },
  created: function () {
    this.fetchFont(this.fontUrl).then((font) => {
      // inject font after creation, because @font-face is not supported by shadow DOM
      let fontFaceSheet = new CSSStyleSheet();
      fontFaceSheet.replaceSync(font);
      document.adoptedStyleSheets = [
        ...document.adoptedStyleSheets,
        fontFaceSheet,
      ];
    });
  },
  methods: {
    async fetchFont(url) {
      const response = await fetch(url);
      return await response.text();
    },
  },
};
</script>
