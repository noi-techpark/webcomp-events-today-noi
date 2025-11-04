<!--
SPDX-FileCopyrightText: NOI Techpark <digital@noi.bz.it>

SPDX-License-Identifier: AGPL-3.0-or-later
-->

<template>
  <div>
    <div id="header">
      <div id="mainTitle"><span class="bold">TODAY</span>.NOI.BZ.IT</div>

      <div id="dateTime">
        <div id="time">{{ options.timestamp }}</div>
        <div id="date">{{ options.currentDate }}</div>
      </div>
    </div>

    <div class="content">
      <div class="line" v-for="event in options.events" :key="event.key">
        <div class="textWidth">
          <a v-if="event.webAddress" :href="event.webAddress" target="_blank">
            <div :class="getTitleClass(event)">
              {{ event.title[options.currentLanguage] }}
            </div>
          </a>
          <div v-else :class="getTitleClass(event)">
            {{ event.title[options.currentLanguage] }}
          </div>
          <div v-if="event.subTitle" class="subTitle fitOneLine spaceToCompany">
            {{ event.subTitle }}
          </div>
          <div class="company">
            {{ event.companyName }}
          </div>
        </div>

        <div class="details">
          <div v-if="event.rooms" class="location">
            <div v-for="(room, index) in event.rooms" :key="room">
              <a
                v-if="event.mapsLinks[index]"
                class="room"
                :href="event.mapsLinks[index]"
                target="_blank"
              >
                <span v-if="index < event.rooms.length - 1"
                  >{{ room }},&nbsp;</span
                >
                <span v-else>{{ room }}</span>
              </a>
              <div v-else>
                <span v-if="index < event.rooms.length - 1"
                  >{{ room }},&nbsp;</span
                >
                <span v-else>{{ room }}</span>
              </div>
            </div>
          </div>

          <div class="starts-in">
            <small>
              {{ event.time }}
            </small>
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
  data: function () {
    return {
      lorem:
        "Lorem ipsum dolor sit amet, consectetur adipiscing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua. Ut enim ad minim veniam, quis nostrud exercitation ullamco laboris nisi ut aliquip ex ea commodo consequat.",
    };
  },
  methods: {
    getTitleClass(event) {
      return {
        title: true,
        spaceToCompany: event.subTitle === "",
        fitOneLine: event.subTitle !== "",
        fitOneLineOverride: event.subTitle === "",
      };
    },
  },
};
</script>

<style>
body {
  width: 100%;
  text-align: center;
  color: #000;
  max-width: 50vw;
  font-size: 11px;
  margin: 0;
  min-height: 100vh;
  height: 100%;
  padding-bottom: 60px;
}

#header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  color: #fff;
  background: #000;
  height: 10.5vh;
}

#dateTime {
  padding-right: 20px;
}

#time {
  font-size: 3.5em;
  font-weight: 700;
}

#date {
  font-size: 2em;
  font-weight: 400;
}

#mainTitle {
  font-size: 5.7em;
  padding-left: 20px;
}

.bold {
  font-weight: 600;
}

.title {
  font-size: 1.9em;
  font-weight: 600;
  text-align: left;
}

.subTitle {
  font-size: 1.4em;
  font-weight: 400;

  text-align: left;
  min-width: 0;
}

.textWidth {
  max-width: 30vw;
  /*  min-width: 0px needed to allow flex container to have size smaller than max-width */
  min-width: 0px;
}

.fitOneLine > div,
a > div {
  overflow: hidden;
  white-space: nowrap;
  text-overflow: ellipsis;
}

.fitOneLineOverride {
  overflow: visible !important;
  white-space: normal !important;
}

.company {
  line-height: 1;
  font-size: 1.4em;
  font-weight: 800;
  color: #8c8c8c;

  text-align: left;
}

.line {
  min-height: 9.05vh;
  padding: 0px 40px;
  display: flex;
  align-items: center;
  justify-content: space-between;
  background: #fff;
  border: solid #000;
  border-width: 0vh 2vh 1vh 2vh;
}

.spaceToCompany {
  margin-bottom: 4px;
}

.details {
  display: flex;
  align-items: center;

  white-space: nowrap;
}

.location {
  color: #fff;
  background: #000;
  padding: 12px 20px;
  font-size: 1.5em;
  font-weight: bold;
  display: flex;
  justify-content: center;
  margin: 15px;
  white-space: normal;
}

/* @media (min-width: 1026px) and (max-width: 1500px) {
  .location {
    font-size: 1.5em;
    white-space: normal;
  }
} */

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
  padding-right: 2.4vh;
  padding-bottom: 10px;
  background: #000;
}

.footer-text {
  color: white;
  display: flex;
  align-items: center;
  justify-content: flex-end;
}
</style>
