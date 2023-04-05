<template>
  <h1>Events for Good</h1>
  <div class="events">
    <EventCard v-for="event in events" :key="event.id" :event="event" />

    <div class="pagination">
      <router-link
        id="page-prev"
        :to="{ name: 'EventList', query: { page: page - 1 } }"
        rel="prev"
        v-if="page != 1"
        >&#60; Previous</router-link
      >

      <router-link
        id="page-next"
        :to="{ name: 'EventList', query: { page: page + 1 } }"
        rel="next"
        v-if="hasNextPage"
        >Next &#62;</router-link
      >
    </div>
  </div>
</template>

<script>
import EventCard from '@/components/EventCard.vue'
import EventService from '@/services/EventService.js'
import NProgress from 'nprogress' 

export default {
  name: 'EventList',
  props: ['page'],
  components: {
    EventCard
  },
  data() {
    return {
      events: null,
      totalEvents: 0
    }
  },
  beforeRouteEnter(routeTo,routeFrom,next) {
    NProgress.start()
      EventService.getEvents(2,parseInt(routeTo.query.page) || 1)
        .then(response => {
          next(comp => { // continue routing and once component is loaded set these values
            comp.events = response.data
            comp.totalEvents = response.headers['x-total-count'] //I'm using comp (as in component in the docs you'll see vm)
          })
         
        })
        .catch(() => {
          next({ name: 'NetworkError' }) // if the API fails, load the NetworkError Page
        }).finally(() =>{
          NProgress.done() // whetheror not the API succeeds or fails,finish the progress bar..
        })
  },

  beforeRouteUpdate(routeTo) {
    NProgress.start()
      EventService.getEvents(2, this.page)
        .then(response => {
            this.events = response.data
            this.totalEvents = response.headers['x-total-count'] //I'm using comp (as in component in the docs you'll see vm)
        })
        .catch(() => {
          next({ name: 'NetworkError' }) // if the API fails, load the NetworkError Page
        }).finally(() =>{
          NProgress.done() // whetheror not the API succeeds or fails,finish the progress bar..
        })
  },






  computed: {
    hasNextPage() {
      var totalPages = Math.ceil(this.totalEvents / 2)

      return this.page < totalPages
    }
  }
}
</script>

<style scoped>
.events {
  display: flex;
  flex-direction: column;
  align-items: center;
}
.pagination {
  display: flex;
  width: 290px;
}
.pagination a {
  flex: 1;
  text-decoration: none;
  color: #2c3e50;
}

#page-prev {
  text-align: left;
}

#page-next {
  text-align: right;
}
</style>
