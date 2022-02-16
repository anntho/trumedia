<template>
  <div id="app">
    <n-config-provider :theme="darkTheme">
      <div class="header">
        <div></div>
        <div class="section">
          <n-select
            placeholder="Select a Player"
            :options="options"
            v-model:value="selected"
            @update:value="selectHandler" />
        </div>
      </div>

      <div v-show="maxAttempts" class="container">
        <div class="inner-container">
          <n-alert title="Error loading data" type="error">Please try again later</n-alert>
        </div>
      </div>

      <div v-show="loaded" class="container">
        <div class="inner-container">
          <n-card :style="{ marginBottom: '20px' }">
            <n-layout has-sider>
              <n-layout-sider>
                <img height="150" :src="player.playerImage">
              </n-layout-sider>

              <n-layout>
                <n-page-header :subtitle="season.header">
                  <n-grid :cols="9">
                    <n-gi><n-statistic label="QBR" :value="season.qbr" /></n-gi>
                    <n-gi><n-statistic label="CMP" :value="season.completions" /></n-gi>
                    <n-gi><n-statistic label="ATT" :value="season.attempts" /></n-gi>
                    <n-gi><n-statistic label="CMP%" :value="season.completionPercentage" /></n-gi>
                    <n-gi><n-statistic label="YDS" :value="season.yards" /></n-gi>
                    <n-gi><n-statistic label="AVG" :value="season.average" /></n-gi>
                    <n-gi><n-statistic label="TD" :value="season.td" /></n-gi>
                    <n-gi><n-statistic label="INT" :value="season.int" /></n-gi>
                    <n-gi><n-statistic label="SACK" :value="season.sack" /></n-gi>
                  </n-grid>
                  <template #title>{{ player.fullName }}</template>
                </n-page-header>
              </n-layout>
            </n-layout>
          </n-card>

          <n-grid x-gap="12" :cols="2" :style="{ marginBottom: '20px' }">
            <n-gi>
              <n-card>
                <Chart :height="300" :data-prop="chartDataCmpPct" />
              </n-card>
            </n-gi>
            <n-gi>
              <n-card>
                <Chart :height="300" :data-prop="chartDataYdAtt" />
              </n-card>
            </n-gi>
          </n-grid>

          <n-card>
            <Table :dataProp="tableData" />
          </n-card>
        </div>
      </div>
    </n-config-provider>
  </div>
</template>

<script>
import _ from 'lodash'
import { darkTheme } from 'naive-ui'
import Chart from './components/Chart'
import Table from './components/Table'
import json from './assets/teams.json'

export default {
  name: 'App',
  components: {
    Chart,
    Table,
  },
  setup() {
    return {
      darkTheme,
    }
  },
  async mounted() {
    await this.fetchPlayers()
  },
  data() {
    return {
      json: json,
      apiToken: process.env.VUE_APP_API_TOKEN,
      apiKey: process.env.VUE_APP_API_KEY,
      attempts: 0,
      maxAttempts: false,
      endpoints: {
        base: 'https://project.trumedianetworks.com',
        token: '/api/token',
        player: '/api/nfl/player',
        players: '/api/nfl/players',
      },
      loaded: false,
      selected: null,
      options: [],
      player: {},
      players: [],
      games: [],
    }
  },
  computed: {
    season() {
      if (_.isEmpty(this.games)) {
        return {}
      }

      const header = `${this.games[0].team} ${this.games[0].seasonYear}`

      const yards = _.sumBy(this.games, game => game.PsYds)
      const passingTD = _.sumBy(this.games, game => game.PsTD)
      const td = _.sumBy(this.games, game => _.add(game.PsTD, game.RshTD))
      const int = _.sumBy(this.games, game => game.Int)
      const sack = _.sumBy(this.games, game => game.Sack)

      const attempts = _.sumBy(this.games, game => game.Att)
      const completions = _.sumBy(this.games, game => game.Cmp)

      const completionPercentage = this.pct(completions, attempts)
      const average = this.avg(yards, attempts)

      const qbr = this.qbr(completions, attempts, yards, passingTD, int)

      return { header, yards, td, int, sack, attempts,
        completions, completionPercentage, average, qbr }
    },
    chartDataCmpPct() {
      if (_.isEmpty(this.games)) {
        return []
      }

      const team = _.find(this.json, j => j.code === this.games[0].team)
      const labels = _.map(this.games, game => this.label(game.opponent, game.gameDate))
      const set_1 = {
        label: 'Cmp%',
        data: _.map(this.games, game => this.pct(game.Cmp, game.Att)),
      }
      const axes = { x: 'Weeks', y: '%' }

      return { team, labels, set_1, axes }
    },
    chartDataYdAtt() {
      if (_.isEmpty(this.games)) {
        return []
      }

      const team = _.find(this.json, j => j.code === this.games[0].team)
      const labels = _.map(this.games, game => this.label(game.opponent, game.gameDate))
      const set_1 = {
        label: 'Yd/Att (Pass)',
        data: _.map(this.games, game => this.avg(game.PsYds, game.Att)),
      }
      const set_2 = {
        label: 'Yd/Att (Rush)',
        data: _.map(this.games, game => this.avg(game.RshYds, game.Rush)),
      }
      const axes = { x: 'Weeks', y: 'Yds' }

      return { team, labels, set_1, set_2, axes }
    },
    tableData() {
      if (_.isEmpty(this.games)) {
        return []
      }

      return _.map(this.games, game => this.createTableObject(game))
    }
  },
  methods: {
    avg(a, b) {
      return b == 0 ? 0 : parseFloat(a/b).toFixed(2)
    },
    pct(a, b) {
      return b == 0 ? 0 : parseFloat((a/b)*100).toFixed(2)
    },
    label(opponent, date) {
      return `${opponent} (${date.substring(5, 10)})`
    },
    dateToString(date) {
      return new Date(date).toDateString()
    },
    qbr(completions, attempts, yards, td, int) {
      if (attempts === 0) { return 0 }

      const a = ((completions / attempts) - .3) * 5
      const b = ((yards / attempts) - 3) * .25
      const c = (td / attempts) * 20
      const d = 2.375 - ((int / attempts) * 25)

      return ((_.add(a, b, c, d) / 6) * 100).toFixed(2)
    },
    createTableObject(game) {
      return {
        date: this.dateToString(game.gameDate),
        week: game.week,
        opp: game.opponent,
        cmp: game.Cmp,
        att: game.Att,
        cmppct: this.pct(game.Cmp, game.Att),
        avg: this.avg(game.PsYds, game.Att),
        pstd: game.PsTD,
        psyds: game.PsYds,
        td: _.add(game.PsTD, game.RshTD),
        int: game.Int,
        rshtd: game.RshTD,
        rshyds: game.RshYds,
        rush: game.Rush,
        sack: game.Sack,
      }
    },

    selectHandler(value) {
      this.player = _.find(this.players, { playerId: value })
      this.fetchPlayerStatistics(value)

      if (!this.loaded) {
        this.loaded = true
      }
    },

    initialSelect() {
      this.selected = this.options[0].value
      this.selectHandler(this.options[0].value)
    },

    async fetchPlayerStatistics(playerId) {
      let endpoint = `${this.endpoints.base}${this.endpoints.player}/${playerId}`
      let playerStatistics = await this.request(endpoint)

      this.games = playerStatistics
    },
    async fetchPlayers() {
      let endpoint = `${this.endpoints.base}${this.endpoints.players}`
      let players = await this.request(endpoint)

      if (!players) {
        return
      }

      this.players = players
      this.options = players.map(player => {
        return {
          value: player.playerId,
          label: player.fullName,
        }
      })

      this.initialSelect()
    },
    async token() {
      let endpoint = `${this.endpoints.base}${this.endpoints.token}`
      let request = await fetch(endpoint, { headers: { 'apiKey': this.apiKey } })
      let response = await request.json()
      this.apiToken = response?.token
    },
    async request(endpoint) {
      let request = await fetch(endpoint, { headers: { 'TempToken': this.apiToken } })
      let response = await request.json()

      if (response.statusCode === 401) {
        if (this.attempts > 1) {
          this.maxAttempts = true
          return
        }

        this.attempts++

        await this.token()

        return await this.request(endpoint)
      }

      return response
    },
  },
}
</script>

<style lang="scss">
  @import "@/styles/app.scss";
</style>
