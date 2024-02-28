<!--
SPDX-FileCopyrightText: NOI Techpark <digital@noi.bz.it>

SPDX-License-Identifier: AGPL-3.0-or-later
-->

<template>
  <body
    :class="getBodyClass"
    v-bind:style="{ 'font-family': this.options.fontName + ', sans-serif' }"
  >
    <div id="header" :class="getHeaderClass">
      <div id="mainTitle" :class="getHeaderPaddingClass">
        <span class="bold">TODAY</span>.NOI.BZ.IT
      </div>
      <img
        v-if="isDefaultTheme"
        src="https://third-party.opendatahub.com/noi-logo/noi-black.svg"
        class="noi-logo"
        alt="NOI logo"
      />
      <div v-else id="dateTime" :class="getHeaderPaddingClass">
        <div id="time">{{ timestamp }}</div>
        <div id="date">{{ currentDate() }}</div>
      </div>
    </div>

    <div class="content">
      <div :class="getLineClass" v-for="event in events" :key="event.key">
        <div>
          <a v-if="event.webAddress" :href="event.webAddress" target="_blank">
            <div :class="getTitleClass">{{ event.title }}</div>
          </a>
          <div v-else :class="getTitleClass">{{ event.title }}</div>
          <div class="subTitle">{{ event.subTitle }}</div>
          <div class="company">{{ event.companyName }}</div>
        </div>

        <div class="details">
          <div class="location">
            <a
              v-if="event.mapsLink"
              class="room"
              :href="event.mapsLink"
              target="_blank"
              >{{ event.room }}</a
            >
            <div v-else>{{ event.room }}</div>
          </div>

          <div class="starts-in">
            <small>
              {{ event.time }}
            </small>
            <br />
            <div class="bold">
              {{ event.startDate }}
            </div>
          </div>
        </div>
      </div>

      <div class="footer">
        <a href="https://opendatahub.com" target="_blank" class="footer-text"
          >powered by Open Data Hub
          <img
            :src="require('@/assets/icons/NOI_OPENDATAHUB_NEW_WH-01.png')"
            height="35px"
        /></a>
      </div>
    </div>
  </body>
</template>

<script>
// define themes
const Themes = Object.freeze({
  default: "default",
  halfScreenBlack: "halfScreenBlack",
  fullScreenBlack: "fullScreenBlack",
});
export default {
  name: "EventsToday",
  props: {
    options: Object,
    default: () => {
      return {};
    },
  },
  data: function () {
    return {
      events: [],
      roomMapping: this.fetchRoomMapping(),
      devMode: this.options.devMode, // shows lorem ipsum as title an subtitle
      theme: this.getTheme(),
      timestamp: "",
      lorem:
        "Lorem ipsum dolor sit amet, consectetur adipiscing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua. Ut enim ad minim veniam, quis nostrud exercitation ullamco laboris nisi ut aliquip ex ea commodo consequat.",
    };
  },
  computed: {
    getHeaderClass() {
      return {
        blackHeader: this.theme != Themes.default,
        halfHeader: this.theme != Themes.halfScreenBlack,
      };
    },
    getBodyClass() {
      return {
        halfScreen: this.theme === Themes.halfScreenBlack,
      };
    },
    getHeaderPaddingClass() {
      return {
        headerPadding: this.theme !== Themes.halfScreenBlack,
      };
    },
    getLineClass() {
      return {
        line: true,
        linePaddingHalfScreen: this.theme === Themes.halfScreenBlack,
      };
    },
    getTitleClass() {
      return {
        title: true,
        titleHalfScreen: this.theme === Themes.halfScreenBlack,
      };
    },
    isDefaultTheme() {
      return this.theme === Themes.default;
    },
  },
  created: function () {
    this.fetchData();
    this.updateTimestamp();

    // cron jobs
    setInterval(this.fetchData, 5 * 60 * 1000); // every 5 minutes
    setInterval(this.updateTimestamp, 1000);
  },
  methods: {
    getTheme() {
      const urlParams = new URLSearchParams(window.location.search);
      const location = urlParams.get("location");

      if (location == null) {
        return Themes.default;
      }

      switch (location.toLocaleLowerCase()) {
        case "foyer":
          return Themes.halfScreenBlack;
        case "d1":
        case "d6":
          return Themes.fullScreenBlack;
        default:
          return Themes.default;
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
          this.options.room == null ||
          this.options.room === "" ||
          element.SpaceDescList.indexOf(this.options.room) > -1
        ) {
          const startDate = new Date(element.RoomStartDate);
          const endDate = new Date(element.RoomEndDate);
          const room = element.SpaceDescList[0];

          let event = {
            title: this.devMode ? this.lorem : element.EventTitle.en,
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
        today.getHours() +
        ":" +
        (today.getMinutes() < 10 ? "0" : "") +
        today.getMinutes();
      this.timestamp = time;
    },
    currentDate() {
      const current = new Date();
      return this.formatDate(current);
    },
  },
};
</script>

<style>
body {
  width: 100%;
  text-align: center;
  color: #000;
  font-size: 16px;
  margin: 0;
  min-height: 100vh;
  height: 100%;
  padding-bottom: 60px;
}

.halfScreen {
  max-width: 50vw;
  font-size: 11px;
}

#header {
  display: flex;
  justify-content: space-between;
  align-items: center;
}

#dateTime {
  padding-right: 20px;
  padding-top: 20px;
}

#time {
  font-size: 3.5em;
  font-weight: 700;
}

#date {
  font-size: 2em;
  font-weight: 400;
}

.blackHeader {
  color: #fff;
  background-color: #000;
}

.halfHeader {
  font-size: 14px;
}

.content {
  border: solid #000 10px;
}

#mainTitle {
  font-size: 5.7em;
  padding-left: 20px;
  padding-top: 20px;
}

.headerPadding {
  padding-bottom: 20px;
}

.bold {
  font-weight: 600;
}

.title {
  font-size: 2.3em;
  font-weight: 600;
  margin-bottom: 8px;
  text-align: left;
  white-space: break-spaces;
}

.titleHalfScreen {
  font-size: 1.9em !important;
}

.subTitle {
  font-size: 1.4em;
  font-weight: 400;
  margin-bottom: 8px;

  text-align: left;

  white-space: break-spaces;
}

.company {
  line-height: 1;
  font-size: 1.4em;
  font-weight: 800;
  color: #8c8c8c;

  text-align: left;

  display: flex;
  justify-content: start;
}

.line {
  min-height: 8em;
  padding: 20px 40px;
  display: flex;
  align-items: center;
  justify-content: space-between;

  border: solid #000 8px;
}

.linePaddingHalfScreen {
  padding: 0px 40px !important;
}

.details {
  display: flex;
  align-items: center;

  white-space: nowrap;
}

.location {
  color: #fff;
  background-color: #000;
  padding: 12px 20px;
  font-size: 1.6em;
  font-weight: bold;
  display: flex;
  justify-content: center;
  margin: 15px;
}

a {
  color: #000;
  text-decoration: underline;
}

a.room {
  color: #ffffff;
}

.noi-logo {
  width: 275px;
}

.starts-in {
  font-size: 2em;
  line-height: 1;
  display: flex;
  flex-direction: column;
}

.footer {
  padding-top: 10px;
  background: #000;
}
.footer-text {
  color: white;
  display: flex;
  align-items: center;
  justify-content: end;
}

@media only screen and (max-width: 1024px) {
  body {
    font-size: 12px;
  }

  #header {
    flex-direction: column;
  }

  #mainTitle {
    font-size: 3em;
  }

  .line {
    flex-direction: column;
    padding: 10px;
  }

  .title {
    max-width: 95vw;
    text-align: center;
  }

  .subTitle {
    max-width: 95vw;
    text-align: center;
  }

  .company {
    justify-content: center;
    text-align: center;
  }

  .details {
    flex-direction: column;
  }

  .location {
    width: 67vw;
  }
}
</style>
