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
        const localizedTitle = {
          en: this.getLocalizedValue(
            {
              en: element.Detail?.en?.Title,
              de: element.Detail?.de?.Title,
              it: element.Detail?.it?.Title,
            },
            "en"
          ),
          de: this.getLocalizedValue(
            {
              en: element.Detail?.en?.Title,
              de: element.Detail?.de?.Title,
              it: element.Detail?.it?.Title,
            },
            "de"
          ),
          it: this.getLocalizedValue(
            {
              en: element.Detail?.en?.Title,
              de: element.Detail?.de?.Title,
              it: element.Detail?.it?.Title,
            },
            "it"
          ),
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
        console.log(eventsParam);

        if (eventsParam != null && eventsParam.toUpperCase() === "NOIBRUNECK") {
          return "NOIBRUNECK";
        }

        return this.eventLocation;
      },
      fetchData() {
        //For testing
        if (this.devMode) {
            const items = [
              {
                Detail: {
                  en: { Title: "English Title" },
                  de: { Title: "" },
                  it: { Title: "Titolo Italiano" },
                },
                Subtitle: "Subtitle 1",
                CompanyName: "Test Company 1",
                EventWebAddress: "",
                RoomStartDate: new Date().getTime(),
                RoomEndDate: new Date().getTime() + 3600000,
                SpaceDescList: ["Room A"],
              },
              {
                Detail: {
                  en: { Title: "" },
                  de: { Title: "Deutscher Titel" },
                  it: { Title: "" },
                },
                Subtitle: "Subtitle 2",
                CompanyName: "Test Company 2",
                EventWebAddress: "",
                RoomStartDate: new Date().getTime(),
                RoomEndDate: new Date().getTime() + 3600000,
                SpaceDescList: ["Room B"],
              },
              {
                Detail: {
                  en: { Title: "" },
                  de: { Title: "" },
                  it: { Title: "" },
                },
                Subtitle: "Subtitle 3",
                CompanyName: "Should not show",
                EventWebAddress: "",
                RoomStartDate: new Date().getTime(),
                RoomEndDate: new Date().getTime() + 3600000,
                SpaceDescList: ["Room C"],
              },
            ];

            this.events = [];

            items.forEach((element) => {
              const startDate = new Date(element.RoomStartDate);
              const endDate = new Date(element.RoomEndDate);

              const localizedFields = this.buildLocalizedFields(element);

              if (
                !localizedFields.title.en &&
                !localizedFields.title.de &&
                !localizedFields.title.it
              ) {
                return;
              }

              let event = {
                title: localizedFields.title,
                subTitle: element.Subtitle,
                companyName: element.CompanyName,
                webAddress: element.EventWebAddress,
                time: this.formatTime(startDate, endDate),
                rooms: element.SpaceDescList.sort(),
                startDate: this.formatDate(startDate),
                mapsLinks: [],
              };

              this.events.push(event);
            });

            return;
          }


        const baseURL =
          "https://tourism.api.opendatahub.com/v1/EventShort/GetbyRoomBooked?";

        const endDate = new Date();
        endDate.setUTCHours(24, 0, 0, 0);

        const params = new URLSearchParams([
          ["startdate", new Date().getTime()],
          ["eventlocation", this.getEventLocation()],
          ["datetimeformat", "uxtimestamp"],
          [
            "publishedon",
            this.getEventLocation() == "NOI" ? "today.noi.bz.it" : "Nobis",
          ],
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

            const localizedFields = this.buildLocalizedFields(element);

            if (
              !localizedFields.title.en &&
              !localizedFields.title.de &&
              !localizedFields.title.it
            ) {
              return;
            }

            let event = {
              title: localizedFields.title,
              subTitle: this.devMode ? this.lorem : element.Subtitle,
              companyName: this.devMode ? this.lorem : element.CompanyName,
              webAddress: element.EventWebAddress,
              time: this.formatTime(startDate, endDate),
              rooms: element.SpaceDescList.sort(),
              startDate: this.formatDate(startDate),
              mapsLinks: element.SpaceDescList.sort().map(
                (r) => this.roomMapping[r]
              ),
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
