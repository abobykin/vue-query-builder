<script>
import uuid from 'uuid';
export default {
  name: 'QueryBuilder',
  props: {
    query: {
      type: Object,
      required: true
    }
  },
  data () {
    return {
      combinators: ['AND', 'OR'],
      operators: ['<', '=', '>', '!='],
      fields: ['Address', 'Twitter'],
      currentQuery: this.query,
    }
  },
  methods: {
    buildOueryObject: function(query) {
      this.currentQuery = query
    },
    findAndRemoveLayer: function(array, layerId, layerType) {
      array.map((el) => {
        switch(layerType) {
          case 'group':
            if (el.id === layerId) {
              this.$delete(el, 'rules')
            }
            if (el.rules) {
              return this.findAndRemoveLayer(el.rules, layerId, layerType)
            }
            break
          case 'row':
            if(el.id === layerId) {
              this.$delete(el, 'field')
            }
            if (el.rules) {
              return this.findAndRemoveLayer(el.rules, layerId, layerType)
            }
            break
          default:
            return
        }
      })
    },
    onRemoveLayer: function(e) {
      const layerType = e.target.getAttribute('data-type')
      const layerId = e.target.getAttribute('data-id')
      this.findAndRemoveLayer(this.currentQuery.rules, layerId, layerType)
      this.updatedQuery()
    },
    addNewLayer: function(array, layerId, layerType) {
      const newGroupObject = {
        id: uuid(),
        rules: [],
        combinator: 'AND'
      }
      const newRuleObject = {
        id: uuid(),
        field: "Address",
        value: "6",
        operator: ">"
      }
      if (layerId == 0) {
        if (layerType === 'group') {
          array.push(newGroupObject)
        }
        if (layerType === 'row') {
          array.push(newRuleObject)
        }
      } else {
        array.map((el) => {
          if (el.id == layerId && !el.rules) {
            if (layerType === 'group') {
              array.push(newGroupObject)
            }
            if (layerType === 'row') {
              array.push(newRuleObject)
            }
          }
          if (el.rules) {
            if (layerType === 'group') {
              el.rules.push(newGroupObject)
            }
            if (layerType === 'row') {
              el.rules.push(newRuleObject)
            }
          }
        })
      }
    },
    onAddRule: function(e) {
      const layerType = e.target.getAttribute('data-type')
      const layerId = e.target.getAttribute('data-id')
      this.addNewLayer(this.currentQuery.rules, layerId, layerType)
      this.updatedQuery()
    },
    updatedQuery: function() {
      this.$emit('updatedQueryEvent', this.currentQuery)
    }
  },
  render() {
    const combinators = this.combinators.map(combinator => {
      return <option>{combinator}</option>
    })
    const operators = this.operators.map(operator => {
      return <option>{operator}</option>
    })
    const fields = this.fields.map(field => {
      return <option>{field}</option>
    })
    const nestedParts = array => {
      return array.map((rule) => {
        if (rule.rules) { return (
          <div class="card mb-3">
            <div class="card-header">
              <div class="form-row">
                <div class="col-md-2">
                  <select v-model={rule.combinator} class="form-control">
                    {combinators}
                  </select>
                </div>
                <div class="col-md-2">
                  <button 
                    onClick={this.onAddRule}
                    data-type="row"
                    data-id={rule.id}
                    type="button" 
                    class="btn btn-outline-secondary btn-block"
                  >
                    + Rule
                  </button>
                </div>
                <div class="col-md-2">
                  <button 
                    onClick={this.onAddRule}
                    data-type="group"
                    data-id={rule.id}
                    type="button" 
                    class="btn btn-outline-secondary btn-block"
                  >
                    + Group
                  </button>
                </div>
                <div class="col-md-1">
                  <button 
                    onClick={this.onRemoveLayer}
                    data-type="group"
                    data-id={rule.id}
                    class="btn btn-outline-secondary"
                  >
                    X
                  </button>
                </div>
              </div>
            </div>
            <div class="card-body text-dark">
              {nestedParts(rule.rules)}
            </div>
          </div>
        )} else if (rule.field) { return (
          <div class="form-row mb-4">
            <div class="col-md-4">
              <select v-model={rule.field} class="form-control">
                {fields} 
              </select>
            </div>
            <div class="col-md-3">
              <select v-model={rule.operator} class="form-control">
                {operators}
              </select>
            </div>
            <div class="col-md-4">
              <input 
                value={rule.value} 
                v-model={rule.value}
                type="text" 
                class="form-control" 
              />
            </div>
            <div class="col-md-1">
              <button 
                onClick={this.onRemoveLayer}
                data-type="row"
                data-id={rule.id}
                class="btn btn-outline-secondary"
              >
                X
              </button>
            </div>
          </div>
        )} else return null
      })
    }
    
    return (
      <div class="card border-dark mb-3 mr-auto ml-auto" style="max-width: 800px">
        <div class="card-header">
          <div class="form-row">
            <div class="col-md-2">
              <select v-model={this.currentQuery.combinator} class="form-control">
                {combinators}
              </select>
            </div>
            <div class="col-md-2">
              <button 
                onClick={this.onAddRule}
                data-type="row"
                data-id={0}
                type="button"
                class="btn btn-outline-secondary btn-block"
              >
                + Rule
              </button>
            </div>
            <div class="col-md-2">
              <button 
                onClick={this.onAddRule}
                data-type="group"
                data-id={0}
                type="button"
                class="btn btn-outline-secondary btn-block"
              >
                + Group
              </button>
            </div>
          </div>
        </div>
        <div class="card-body text-dark">
          {this.currentQuery.rules && nestedParts(this.currentQuery.rules)}
        </div>
      </div>      
    )
  },
  created() {
    this.buildOueryObject(this.query)
  },
  updated() {
    this.buildOueryObject(this.currentQuery)
  }
}
</script>

<style scoped></style>
