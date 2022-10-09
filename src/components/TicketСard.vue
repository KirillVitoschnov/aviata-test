<template>
  <div class="ticket-card-wrapper">
    <div class="meta-wrapper">
      <div class="ticket-info">
        <div class="avia-logo">
          <img :src="`https://aviata.kz/static/airline-logos/80x80/${ticketData.validating_carrier}.png`">
          <div>{{ avia[ticketData.validating_carrier] }}</div>
        </div>
        <div class="avia-date-main-wrapper">
          <div class="avia-date-wrapper" v-html="segmentsInfo.startDate">

          </div>
          <div class="avia-date-chart">
            <div class="avia-date-line">
              <div class="avia-date-dot">
                <div class="city-text">{{ segmentsInfo.aeroStart }}</div>
              </div>
              <div class="avia-date-time">{{ segmentsInfo.travelTime }}</div>
              <div v-if="segmentsInfo.betweenCity" class="avia-date-city">Через {{ segmentsInfo.betweenCity }}
                {{ segmentsInfo.betweenTime }}
              </div>
              <div v-if="segmentsInfo.betweenCity" class="avia-date-dot"></div>
              <div class="avia-date-dot">
                <div class="city-text">{{ segmentsInfo.aeroEnd }}</div>
              </div>
            </div>
          </div>
          <div class="avia-date-wrapper" v-html="segmentsInfo.endDate"></div>
        </div>
      </div>
      <div class="ticket-info-additional">
        <button class="clear-filters ticket-url">Детали перелета</button>
        <button class="clear-filters ticket-url">Условия тарифа</button>
        <div v-if="!ticketData.refundable" class="ticket-no-refundable">
          <svg xmlns="http://www.w3.org/2000/svg" width="16" height="16" viewBox="0 0 20 20" fill="none">
            <path fill-rule="evenodd" clip-rule="evenodd"
                  d="M4.62478 4.63246L2.18383 5.00012L2 3.77965L5.66142 3.22815L6.21291 6.88957L4.99244 7.0734L4.62478 4.63246Z"
                  fill="#707276"/>
            <path fill-rule="evenodd" clip-rule="evenodd"
                  d="M5.50024 3.53114C3.45478 4.96005 2.1168 7.33197 2.1168 10.0164C2.1168 14.3824 5.65618 17.9218 10.0222 17.9218C14.3883 17.9218 17.9277 14.3824 17.9277 10.0164C17.9277 5.66296 14.4087 2.13143 10.0601 2.11104L10.035 3.11096C13.8429 3.11787 16.9277 6.20689 16.9277 10.0164C16.9277 13.8302 13.836 16.9218 10.0222 16.9218C6.20847 16.9218 3.1168 13.8302 3.1168 10.0164C3.1168 7.8267 4.13598 5.87505 5.72584 4.60994L5.50024 4.61574V3.99863V3.53114Z"
                  fill="#707276"/>
            <path
                d="M17.4363 1.88861L17.0851 1.53274L16.7292 1.88395L1.64918 16.7664L1.2933 17.1176L1.64451 17.4735L2.51148 18.352L2.86269 18.7078L3.21857 18.3566L18.2986 3.47418L18.6545 3.12296L18.3032 2.76709L17.4363 1.88861Z"
                fill="#707276" stroke="white"/>
          </svg>
          <span>невозвратный</span>
        </div>
      </div>


    </div>
    <div class="price-wrapper">
      <h1>{{ ticketData.price }} ₸</h1>
      <button class="price-wrapper-button">Выбрать</button>
      <span class="price-wrapper-passenger-text">Цена за всех пасажиров</span>
      <div class="price-wrapper-additional">
        <span class="bagage-text">
          {{ ticketData.services ? ticketData.services[0].alt_text : '' }}
        </span>
        <button><span>Добавить багаж</span></button>
      </div>
    </div>
  </div>
</template>
<script>
export default {
  name: 'ticketCard',
  props: {
    ticket: Object,
    avia: Object
  },
  data() {
    return {
      ticketData: {},
      segmentsInfo: {
        travelTime: '',
        aeroStart: '',
        aeroEnd: '',
        betweenCity: '',
        betweenTime: '',
        startDate: '',
        endDate: ''
      }
    }
  },
  methods: {
    updateData() {
      this.ticketData = JSON.parse(JSON.stringify(this.ticket))
      this.ticketData.services = Object.values(this.ticketData.services)

      function secondsToHms(d) {
        d = Number(d);
        let h = Math.floor(d / 3600);
        let m = Math.floor(d % 3600 / 60);
        let hDisplay = h > 0 ? h + ' ч ' : "";
        let mDisplay = m > 0 ? m + ' м' : "";
        return hDisplay + mDisplay;
      }

      function timeResult(startDate, endDate) {
        let start = new Date(startDate);
        let end = new Date(endDate);
        let diff = Date.parse(end) - Date.parse(start);
        let days = Math.floor(diff / (1000 * 3600 * 24));
        let hours = Math.floor((diff / (1000 * 3600)) % 24);
        let minutes = Math.floor((diff / 1000 / 60) % 60);
        return (days > 0 ? days + ' д ' : '') + (hours > 0 ? hours + ' ч ' : '') + (minutes > 0 ? minutes + ' м' : '')
      }


      this.segmentsInfo.travelTime = secondsToHms(this.ticketData.itineraries[0][0].traveltime)
      if (this.ticketData.itineraries[0][0].segments.length == 1) {
        let data = this.ticketData.itineraries[0][0].segments[0]
        this.segmentsInfo.startDate = `<span class="avia-date">${data.dep_time.slice(0, -5)}</span><span class="avia-time">${data.dep_time.slice(-5)}</span>`
        this.segmentsInfo.endDate = `<span class="avia-date">${data.arr_time.slice(0, -5)}</span><span class="avia-time">${data.arr_time.slice(-5)}</span>`
        this.segmentsInfo.aeroStart = data.dest_code
        this.segmentsInfo.aeroEnd = data.origin_code
      } else {
        let data = this.ticketData.itineraries[0][0].segments
        this.segmentsInfo.startDate = `<span class="avia-date">${data[0].dep_time.slice(0, -5)}</span><span class="avia-time">${data[0].dep_time.slice(-5)}</span>`
        this.segmentsInfo.endDate = `<span class="avia-date">${data[1].arr_time.slice(0, -5)}</span><span class="avia-time">${data[1].arr_time.slice(-5)}</span>`
        this.segmentsInfo.aeroStart = data[0].origin_code
        this.segmentsInfo.betweenCity = data[0].dest.slice(-1) == 'а' ? data[0].dest.slice(0, -1) + 'у' : data[0].dest
        this.segmentsInfo.aeroEnd = data[1].dest_code
        this.segmentsInfo.betweenTime = timeResult(data[0].dep_time_iso, data[0].arr_time_iso)
      }
    }
  },
  mounted() {
    this.updateData()
  },
  watch: {
    ticket() {
      this.updateData()
    }
  }
}
</script>
