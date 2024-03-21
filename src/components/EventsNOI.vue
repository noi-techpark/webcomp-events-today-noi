<!--
SPDX-FileCopyrightText: NOI Techpark <digital@noi.bz.it>

SPDX-License-Identifier: AGPL-3.0-or-later
-->

<template>
  <div>
    <div id="header">
      <div id="mainTitle"><span class="bold">TODAY</span>.NOI.BZ.IT</div>
      <img
        src="https://third-party.opendatahub.com/noi-logo/noi-black.svg"
        class="noi-logo"
        alt="NOI logo"
      />
    </div>

    <div class="content">
      <div class="line" v-for="event in options.events" :key="event.key">
        <div class="textWidth">
          <a v-if="event.webAddress" :href="event.webAddress" target="_blank">
            <div class="title">
              {{ event.title[options.currentLanguage] }}
            </div>
          </a>
          <div v-else class="title">
            {{ event.title[options.currentLanguage] }}
          </div>
          <div v-if="event.subTitle" class="subTitle">
            {{ event.subTitle }}
          </div>
          <div class="company">
            {{ event.companyName }}
          </div>
        </div>

        <div class="details">
          <div v-if="event.room" class="location">
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
};
</script>

<style>
body {
  width: 100%;
  font-size: 16px;
  text-align: center;
  color: #000;
  margin: 0;
  min-height: 100vh;
  height: 100%;
  padding-bottom: 60px;
}

#header {
  display: flex;
  justify-content: space-between;
  align-items: center;
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
}

.subTitle {
  font-size: 1.4em;
  font-weight: 400;
  margin-bottom: 8px;

  text-align: left;
  min-width: 0;
}

.textWidth {
  max-width: 75vw;
  /*  min-width: 0px needed to allow flex container to have size smaller than max-width */
  min-width: 0px;
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
  padding: 20px 40px;
  display: flex;
  align-items: center;
  justify-content: space-between;
  background-color: #fff;
  border: solid #000;
  border-width: 1.6vh 2.4vh 0vh 2.4vh;
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
