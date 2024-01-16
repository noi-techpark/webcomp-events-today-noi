<!--
SPDX-FileCopyrightText: NOI Techpark <digital@noi.bz.it>

SPDX-License-Identifier: AGPL-3.0-or-later
-->

<template>
  <body
    v-bind:style="{ 'font-family': this.options.fontName + ', sans-serif' }"
  >
    <header>
      <div id="mainTitle"><span class="bold">TODAY</span>.NOI.BZ.IT</div>
      <img
        src="https://third-party.opendatahub.com/noi-logo/noi-black.svg"
        class="noi-logo"
        alt="NOI logo"
      />
    </header>

    <div class="content">
      <div class="row line" v-for="event in events" :key="event.key">
        <div>
          <a v-if="event.webAddress" :href="event.webAddress" target="_blank">
            <div class="title">{{ event.title }}</div>
          </a>
          <div v-else class="title">{{ event.title }}</div>
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
      devMode: this.options.devMode, // shows lorem ipsum as title an subtitle
      lorem:
        "Lorem ipsum dolor sit amet, consectetur adipiscing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua. Ut enim ad minim veniam, quis nostrud exercitation ullamco laboris nisi ut aliquip ex ea commodo consequat.",
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
          title: this.devMode ? this.lorem : element.EventTitle.en,
          subTitle: this.devMode ? this.lorem : element.Subtitle,
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
  padding-bottom: 20px;
}

header {
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.content {
  border: solid #000 10px;
}

#mainTitle {
  font-size: 5rem;
  padding: 20px;
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
  max-width: 70vw;
}

.subTitle {
  font-size: 1.4rem;
  font-weight: 400;
  margin-bottom: 8px;

  text-align: left;

  white-space: break-spaces;
  max-width: 70vw;
}

.company {
  line-height: 1;
  font-size: 1.4rem;
  font-weight: 800;
  color: #8c8c8c;

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

.details {
  display: flex;
  align-items: center;
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

  #mainTitle {
    font-size: 3rem;
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
  }

  .details {
    flex-direction: column;
  }

  .location {
    width: 67vw;
  }

  header {
    flex-direction: column;
  }
}
</style>
