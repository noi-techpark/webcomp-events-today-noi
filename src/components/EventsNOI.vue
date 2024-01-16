<!--
SPDX-FileCopyrightText: NOI Techpark <digital@noi.bz.it>

SPDX-License-Identifier: AGPL-3.0-or-later
-->

<template>
  <div
    id="base"
    v-bind:style="{ 'font-family': this.options.fontName + ', sans-serif' }"
  >
    <header>
      <h1 id="mainTitle"><strong>TODAY</strong>.NOI.BZ.IT</h1>
      <img
        :src="require('@/assets/icons/NOI_2_BK_borderless.png')"
        class="noi-logo"
      />
    </header>

    <div class="content">
      <div class="row line" v-for="event in events" :key="event.key">
        <div>
          <a v-if="event.webAddress" :href="event.webAddress" target="_blank">
            <strong class="title">{{ event.title }}</strong>
          </a>
          <div v-else class="title">{{ event.title }}</div>
          <div class="subTitle">{{ event.subTitle }}</div>
          <div class="company">{{ event.companyName }}</div>
        </div>

        <div class="right">
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
            <div>
              <small>
                {{ event.time }}
              </small>
              <br />
              <strong>
                {{ event.startDate }}
              </strong>
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
  </div>
</template>

<script>
export default {
  name: "EventsToday",
  props: {
    options: Object,
    default: () => {
      return {};
    },
  },
  created: function () {
    this.fetchData();
  },
  data: function () {
    return {
      events: [],
      roomMapping: this.fetchRoomMapping(),
      devMode: false,
      lorem:
        "Lorem ipsum dolor sit amet, consectetur adipiscing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua. Ut enim ad minim veniam, quis nostrud exercitation ullamco laboris nisi ut aliquip ex ea commodo consequat. Duis aute irure dolor in reprehenderit in voluptate velit esse cillum dolore eu fugiat nulla pariatur. Excepteur sint occaecat cupidatat non proident, sunt in culpa qui officia deserunt mollit anim id est laborum.",
    };
  },
  methods: {
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
      for (let i = 0; i <= items.length - 1; i++) {
        const element = items[i];
        const startDate = new Date(element.RoomStartDate);
        const endDate = new Date(element.RoomEndDate);
        const room = element.SpaceDescList[0];

        let event = {
          title: this.devMode
            ? { en: this.lorem, it: this.lorem, de: this.lorem }
            : element.EventTitle.en,
          subTitle: this.devMode
            ? { en: this.lorem, it: this.lorem, de: this.lorem }
            : element.Subtitle,
          companyName: element.CompanyName,
          webAddress: element.EventWebAddress,
          time: this.formatTime(startDate, endDate),
          room: room,
          startDate: this.formatDate(startDate),
          mapsLink: this.roomMapping[room],
        };
        this.events.push(event);
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
        startDate.getHours() +
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
  },
};
</script>

<style>
#base {
  width: 100%;
  text-align: center;
  color: #000;
  font-size: 16px;
  margin: 0;
  min-height: 100vh;
  height: 100%;
  padding-bottom: 20px;
}

header {
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.content {
  box-sizing: content-box;
  border: solid #000 10px;
}

#mainTitle {
  font-size: 5em;
  padding: 25px;
  margin: 0;
}

.title {
  font-size: 2.3em;
}

.company {
  line-height: 1;
  font-weight: bold;
  color: #8c8c8c;
  float: left;
}

.line {
  min-height: 6em;
  padding: 0 20px;
  display: flex;
  align-items: center;
  justify-content: space-between;

  box-sizing: content-box;
  border: solid #000 5px;
}

.right {
  display: flex;
  align-items: center;
}

.location {
  color: #fff;
  background-color: #000;
  padding: 10px 20px;
  font-size: 1.6em;
  font-weight: bold;
  display: flex;
  align-items: center;
}

a {
  color: #000;
  text-decoration: underline;
}

a.room {
  color: #ffffff;
}

.noi-logo {
  width: 155px;
  margin: 20px;
}

.starts-in {
  text-align: right;
  font-size: 2em;
  line-height: 1;
  justify-content: right;
}

.starts-in strong {
  font-size: 1.25em;
}
.footer {
  padding-top: 20px;
  bottom: 0;
  left: 0;
  width: 100%;
  text-align: right;
  z-index: 100001;
}
.footer-text {
  color: white;
}
</style>
