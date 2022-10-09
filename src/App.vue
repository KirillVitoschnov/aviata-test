<template>
  <div id="app">
    <div class="container">
      <div class="main-wrapper">
        <div class="filter-column">
          <options-filter v-model="filterOptions" :input="filterOptions" :options="tariffFilter"
                          :title="'Опции тарифа'"></options-filter>
          <options-filter v-model="filterAvia" :input="filterAvia" :options="data.airlines"
                          :title="'Авиакомпании'"></options-filter>
          <button class="clear-filters" @click="filterOptions=[];filterAvia=[]">Сбросить все фильтры</button>
        </div>
        <div class="list-column">
          <TransitionGroup name="tickets" tag="div">
          <ticket-card :avia="data.airlines" :ticket="ticket" v-for="ticket in data.flights"
                       :key="ticket.id"></ticket-card>
          </TransitionGroup>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import json from '../public/results.json'
import OptionsFilter from './components/OptionsFilter.vue'
import ticketCard from './components/TicketСard.vue'

export default {
  name: 'App',
  components: {
    ticketCard,
    OptionsFilter,
  },
  data() {
    return {
      filterOptions: [],
      filterAvia: [],
      tariffFilter: {
        'straight': 'Только прямые',
        'baggage': 'Только с багажом',
        'returnable': 'Только возвратные'
      },
      data: {
        airlines: {},
        flights: []
      }
    }
  },
  mounted() {
    this.data = JSON.parse(JSON.stringify(json))
  },
  methods: {
    aviaCompanyFilter() {
      this.data = JSON.parse(JSON.stringify(json))
      if (this.filterAvia.length) {
        this.data.flights = this.data.flights.filter(({validating_carrier}) => this.filterAvia.includes(validating_carrier));
      }
      if (this.filterOptions.length) {
        if (this.filterOptions.includes('returnable')) {
          this.data.flights = this.data.flights.filter(function (e) {
            return e.refundable;
          });
        }
        if (this.filterOptions.includes('baggage')) {
          this.data.flights = this.data.flights.filter(function (e) {
            return !e.services['0PC'];
          });
        }
        if (this.filterOptions.includes('straight')) {
          this.data.flights = this.data.flights.filter(function (e) {
            return e.itineraries[0][0].segments.length == 1;
          });
        }
      }
    }

  },
  watch: {
    filterAvia() {
      this.aviaCompanyFilter()
    },
    filterOptions() {
      this.aviaCompanyFilter()
    }
  }
}
</script>


