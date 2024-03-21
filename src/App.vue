<!--
SPDX-FileCopyrightText: NOI Techpark <digital@noi.bz.it>

SPDX-License-Identifier: AGPL-3.0-or-later
-->

<template>
  <body v-bind:style="{ 'font-family': fontName + ', sans-serif' }">
    <EventsNOIFoyer
      v-if="theme === 'foyer'"
      :options="{
        devMode: devMode,
        events: events,
        currentLanguage: currentLanguage,
        currentDate: currentDate(),
        timestamp: timestamp,
      }"
    />
    <EventsNOID1
      v-else-if="theme === 'd1'"
      :options="{
        devMode: devMode,
        events: events,
        currentLanguage: currentLanguage,
        timestamp: timestamp,
        currentDate: currentDate(),
      }"
    />
    <EventsNOI
      v-else
      :options="{
        devMode: devMode,
        events: events,
        currentLanguage: currentLanguage,
      }"
    />
  </body>
</template>
<script>
import EventsNOI from "./components/EventsNOI.vue";
import EventsNOID1 from "./components/EventsNOID1.vue";
import EventsNOIFoyer from "./components/EventsNOIFoyer.vue";

export default {
  name: "App",
  props: {
    room: { type: String, default: "" },
    fontUrl: {
      type: String,
      default: "https://fonts.testingmachine.eu/source-sans-pro/style.css",
    },
    languageRotationInterval: {
      type: Number,
      default: 10,
    },
    fontName: {
      type: String,
      default: "Source Sans Pro",
    },
    devMode: {
      type: Boolean,
      default: false,
    },
  },
  data: function () {
    return {
      events: [],
      roomMapping: this.fetchRoomMapping(),
      theme: this.getTheme(),
      timestamp: "",
      languages: ["en", "de", "it"],
      currentLanguage: "en",
      lorem:
        "Lorem ipsum dolor sit amet, consectetur adipiscing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua. Ut enim ad minim veniam, quis nostrud exercitation ullamco laboris nisi ut aliquip ex ea commodo consequat.",
    };
  },
  components: {
    EventsNOI,
    EventsNOID1,
    EventsNOIFoyer,
  },
  created: function () {
    this.fetchData();
    this.updateTimestamp();

    // font
    this.fetchFont(this.fontUrl).then((font) => {
      // inject font after creation, because @font-face is not supported by shadow DOM
      let fontFaceSheet = new CSSStyleSheet();
      fontFaceSheet.replaceSync(font);
      document.adoptedStyleSheets = [
        ...document.adoptedStyleSheets,
        fontFaceSheet,
      ];
    });

    // cron jobs
    setInterval(this.fetchData, 5 * 60 * 1000); // every 5 minutes
    setInterval(this.updateTimestamp, 2000);
    setInterval(this.rotateLanguage, this.languageRotationInterval * 1000);
  },

  methods: {
    async fetchFont(url) {
      const response = await fetch(url);
      return await response.text();
    },
    getTheme() {
      const urlParams = new URLSearchParams(window.location.search);
      const location = urlParams.get("location");

      if (location == null) {
        return "default";
      }

      switch (location.toLocaleLowerCase()) {
        case "foyer":
          return "foyer";
        case "d1":
        case "d6":
          return "d1";
        default:
          return "default";
      }
    },
    fetchData() {
      const baseURL =
        "https://tourism.api.opendatahub.com/v1/EventShort/GetbyRoomBooked?";

      const endDate = new Date();
      endDate.setUTCHours(24, 0, 0, 0);

      const params = new URLSearchParams([
        ["startdate", new Date().getTime()],
        ["eventlocation", "NOI"],
        ["datetimeformat", "uxtimestamp"],
        ["onlyactive", true],
        ["sortorder", "ASC"],
        ["origin", "webcomp-events-today-noi"],
      ]);

      const xhttp = new XMLHttpRequest();
      xhttp.open("GET", baseURL + params, false);
      xhttp.send();

      this.events = [];

      const items = JSON.parse(xhttp.response);
      items.forEach((element) => {
        if (
          this.room == null ||
          this.room === "" ||
          element.SpaceDescList.indexOf(this.room) > -1
        ) {
          const startDate = new Date(element.RoomStartDate);
          const endDate = new Date(element.RoomEndDate);
          const room = element.SpaceDescList[0];

          let event = {
            title: this.devMode
              ? { en: this.lorem, it: this.lorem, de: this.lorem }
              : element.EventTitle,
            subTitle: this.devMode ? this.lorem : element.Subtitle,
            companyName: this.devMode ? this.lorem : element.CompanyName,
            webAddress: element.EventWebAddress,
            time: this.formatTime(startDate, endDate),
            room: room,
            startDate: this.formatDate(startDate),
            mapsLink: this.roomMapping[room],
          };
          this.events.push(event);
        }
      });
    },
    fetchRoomMapping() {
      const baseURL =
        "https://tourism.opendatahub.com/v1/EventShort/RoomMapping?language=en";

      const xhttp = new XMLHttpRequest();
      xhttp.open("GET", baseURL, false);
      xhttp.send();

      return JSON.parse(xhttp.response);
    },

    formatTime(startDate, endDate) {
      return String(
        String(startDate.getHours()).padStart(2, "0") +
          ":" +
          String(startDate.getMinutes()).padStart(2, "0") +
          " - " +
          endDate.getHours() +
          ":" +
          String(endDate.getMinutes()).padStart(2, "0")
      );
    },
    rotateLanguage() {
      let index = this.languages.indexOf(this.currentLanguage) + 1;
      if (index >= this.languages.length) index = 0;
      this.currentLanguage = this.languages[index];
    },
    formatDate(date) {
      let options = {
        year: "2-digit",
        month: "short",
        day: "2-digit",
      };
      return date.toLocaleDateString("en-GB", options);
    },
    updateTimestamp() {
      const today = new Date();
      const time =
        String(today.getHours()).padStart(2, "0") +
        ":" +
        String(today.getMinutes()).padStart(2, "0");
      this.timestamp = time;
    },
    currentDate() {
      const current = new Date();
      return this.formatDate(current);
    },
  },
};
</script>
