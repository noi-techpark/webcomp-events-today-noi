<!--
SPDX-FileCopyrightText: NOI Techpark <digital@noi.bz.it>

SPDX-License-Identifier: AGPL-3.0-or-later
-->

<template>
  <body v-bind:style="{ 'font-family': fontName + ', sans-serif' }">
    <EventsNOIFoyer
      v-if="theme === 'foyer'"
      :options="{
        events: events,
        currentLanguage: currentLanguage,
        currentDate: currentDate(),
        timestamp: timestamp,
      }"
    />
    <EventsNOID1
      v-else-if="theme === 'd1'"
      :options="{
        events: events,
        currentLanguage: currentLanguage,
        timestamp: timestamp,
        currentDate: currentDate(),
      }"
    />
    <EventsNOI
      v-else
      :options="{
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

import config from "./components/config.js";

export default {
  name: "App",
  props: {
    eventLocation: { type: String, default: "NOI" },
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
  },
  data: function () {
    return {
      events: [],
      roomMapping: this.fetchRoomMapping(),
      theme: this.getTheme(),
      timestamp: "",
      languages: ["en", "de", "it"],
      currentLanguage: "en",
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
    getLanguageFallbackOrder(language) {
      const fallbackOrder = {
        en: ["en", "it", "de"],
        de: ["de", "it", "en"],
        it: ["it", "de", "en"],
      };
      return fallbackOrder[language] || fallbackOrder.en;
    },

    getLocalizedValue(values, language) {
      if (!values || typeof values !== "object") {
        return null;
      }

      const order = this.getLanguageFallbackOrder(language);

      for (const lang of order) {
        const value = values[lang];
        if (typeof value === "string" && value.trim() !== "") {
          return value.trim();
        }
      }

      return null;
    },

    buildLocalizedFields(element) {
      const titles = {
        en: element.Detail?.en?.Title,
        de: element.Detail?.de?.Title,
        it: element.Detail?.it?.Title,
      };
      const localizedTitle = {
        en: this.getLocalizedValue(titles, "en"),
        de: this.getLocalizedValue(titles, "de"),
        it: this.getLocalizedValue(titles, "it"),
      };

      return {
        title: localizedTitle,
      };
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
    // overrides eventLocation prop, if events param is set in the url
    // https://today.noi.bz.it/?events=noibruneck
    getEventLocation() {
      const urlParams = new URLSearchParams(window.location.search);
      const eventsParam = urlParams.get("events");

      if (eventsParam != null && eventsParam.toUpperCase() === "NOIBRUNECK") {
        return "NOIBRUNECK";
      }

      return this.eventLocation;
    },
    fetchData() {
      const baseURL = config.API_BASE_URL + "/Event?";

      const params = new URLSearchParams([
        ["tagfilter", this.getEventLocation() === "NOI" ? "noi" : "noibruneck"],
        ["begindate", Date.now().toString()],
        ["datetimeformat", "uxtimestamp"],
        ["sort", "upcomping"],
        ["active", "true"],
        [
          "publishedon",
          this.getEventLocation() === "NOI" ? "today.noi.bz.it" : "nobis",
        ],
        ["pagesize", "0"],
        ["optimizedates", "true"],
        ["origin", "webcomp-events-today-noi"],
        ["denormalize", "true"],
      ]);

      const xhttp = new XMLHttpRequest();
      xhttp.open("GET", baseURL + params, false);
      xhttp.send();
      const items = (JSON.parse(xhttp.response).Items || []).flat();

      // obtaining the list of venues of NOI
      const VenueURL =
        this.getEventLocation() === "NOI"
          ? config.API_BASE_URL + "/Venue?&source=noi"
          : config.API_BASE_URL + "/Venue?&source=nobis";

      const xhttpVenue = new XMLHttpRequest();
      xhttpVenue.open("GET", VenueURL, false);
      xhttpVenue.send();

      //Chosing the venues of NOI Bz or NOI Bruneck
      const Venues =
        this.getEventLocation() === "NOI"
          ? JSON.parse(xhttpVenue.response).Items[
              JSON.parse(xhttpVenue.response).Items.findIndex(
                (r) =>
                  r.Id === "urn:venue:noi:6b3f0a14-3c5b-5d09-81f3-3ebe5b7885ea"
              )
            ]
          : JSON.parse(xhttpVenue.response).Items[
              JSON.parse(xhttpVenue.response).Items.findIndex(
                (r) =>
                  r.Id ===
                  "urn:venue:noibruneck:aa571d89-36c1-50fa-9400-7f15b1ae814b"
              )
            ];

      this.events = [];
      items.forEach((element) => {
        if (
          this.room == null ||
          this.room === "" ||
          element.EvenDate[0].VenueRoomDetailsIds.indexOf(this.room) > -1
        ) {
          const startDate = new Date(element.EventDate[0].FromUTC);
          const endDate = new Date(element.EventDate[0].ToUTC);

          const localizedFields = this.buildLocalizedFields(element);

          if (
            !localizedFields.title.en &&
            !localizedFields.title.de &&
            !localizedFields.title.it
          ) {
            localizedFields.title = {
              en: "No title",
              de: "Kein Titel",
              it: "Nessun titolo",
            };
          }

          //Creation of the event
          let event = {
            title: localizedFields.title,
            subTitle: element.EventDate[0].EventDateAdditionalInfo
              ? element.EventDate[0].EventDateAdditionalInfo?.en.Description
              : null,
            companyName: element.OrganizerInfos
              ? element.OrganizerInfos.en.CompanyName
              : null,
            webAddress: element.EventUrls ? element.EventUrls[0].Url.en : null,
            time: this.formatTime(startDate, endDate),
            rooms: element.EventDate.map((ed) => ed.VenueRoomDetailsIds)
              .flat()
              .map((room) => {
                return Venues.RoomDetails[
                  Venues.RoomDetails.findIndex((r) => r.Id === room)
                ].Shortname;
              })
              .map((r) => r.replace("NOI ", "")),
            startDate: this.formatDate(startDate),
            mapsLinks: element.EventDate.map((ed) => ed.VenueRoomDetailsIds)
              .flat()
              .map((room) => {
                return Venues.RoomDetails[
                  Venues.RoomDetails.findIndex((r) => r.Id === room)
                ].Shortname;
              })
              .map((r) => r.replace("NOI ", ""))
              .map((r) => this.roomMapping[r]),
          };
          this.events.push(event);
        }
      });

      // if no events are present, add "no events happening right now" placeholder
      if (this.events.length == 0) {
        let placeholderEvent = {
          title: {
            en: "No events",
            it: "Non ci sono eventi",
            de: "Keine Events",
          },
          companyName: "",
          time: "",
          room: "",
        };
        this.events.push(placeholderEvent);
      }
    },
    fetchRoomMapping() {
      const baseURL =
        config.API_BASE_URL +
        "/Venue?idlist=urn:venue:noi:6b3f0a14-3c5b-5d09-81f3-3ebe5b7885ea&denormalize=true&fields=RoomDetails.[*].Shortname,RoomDetails.[*].Mapping.maps.roommapping";

      const xhttp = new XMLHttpRequest();
      xhttp.open("GET", baseURL, false);
      xhttp.send();

      const result = {};

      const items = JSON.parse(xhttp.response).Items;

      items.forEach((item) => {
        const shortname = item["RoomDetails.[*].Shortname"];
        const mapping = item["RoomDetails.[*].Mapping.maps.roommapping"];

        if (shortname && mapping) {
          result[shortname] = mapping;
        }
      });
      return result;
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
