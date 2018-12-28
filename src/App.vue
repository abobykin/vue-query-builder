<template>
  <QueryBuilder 
    :query="query" 
    @updatedQueryEvent="queryUpdated = $event" 
    ref="child" 
  />
</template>

<script>
import QueryBuilder from './QueryBuilder.vue'

let initialQuery = {
  id: "001",
  rules: [
    {
      id: '002',
      field: "Address",
      value: "6",
      operator: "="
    },
    {
      id: '003',
      rules: [
        {
          id: '004',
          field: "Twitter",
          value: "5",
          operator: "<"
        }
      ],
      combinator: 'OR'
    }
  ],
  combinator: 'AND'
}

export default {
  data() {
    return {
      query: initialQuery,
      queryUpdated: {}
    }
  },
  components: {
    QueryBuilder
  },
  methods: {
    eventHandler(updatedQueryEvent) {
      // Here is the result in ordinary object and in JSON
      console.log(this.queryUpdated)
      console.log(JSON.stringify(this.queryUpdated))
    }
  },
  mounted() {
    this.$refs.child.$on('updatedQueryEvent', this.eventHandler);
  }
}
</script>

<style></style>
