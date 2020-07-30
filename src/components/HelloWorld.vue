<template>
  <v-container>
    <v-row>
      <v-col v-for="category in categories" :key="category">
        <h3>{{ category }}</h3>
        <div v-for="(field, key) in getCategoryFields(category)" :key="key" class="d-flex">
          <div class="flex-grow-1">
            <v-tooltip top>
              <template v-slot:activator="{ on, attrs }">
                <v-text-field
                        :label="field.label"
                        :type="field.type"
                        :readonly="field.readOnly"
                        :value="getConstantValue(key)"
                        @change="updateValue($event, key)"
                        v-bind="attrs"
                        v-on="on"
                ></v-text-field>
              </template>
              <span>{{ field.label }} : {{ getConstantValue(key) }}</span>
            </v-tooltip>
          </div>
          <v-tooltip v-if="getChanges(key) && getChanges(key).change !== 0" top>
            <template v-slot:activator="{ on, attrs }">
              <v-icon class="align-self-start"
                      :class="{'red--text': getChanges(key).change > 0, 'green--text': getChanges(key).change < 0}"
                      v-bind="attrs"
                      v-on="on"
              >{{ getChanges(key).change > 0 ? 'mdi-arrow-up' : 'mdi-arrow-down'}}</v-icon>
            </template>
            <span>{{ Math.abs(getChanges(key).change).toFixed(2) }}</span>
          </v-tooltip>
        </div>
      </v-col>
    </v-row>
  </v-container>
</template>

<script>
  import constantsData from './constants.json';

  function getComputed() {
    let computed = {};
    Object.keys(constantsData.fields).forEach(eachField => {
      if (constantsData.fields[eachField].calculate) {
        let computeString = constantsData.fields[eachField].calculate;
        computed[eachField] = () => {
          return eval(computeString);
        }
      }
    });
    return computed;
  }

  export default {
    name: 'HelloWorld',

    data: () => ({
      initialized: false,
      constants: {},
      categories: [],
      changes: [],
    }),

    computed: getComputed(),

    mounted() {
      this.processConstants();
    },

    methods: {
      processConstants() {
        let vm = this;
        Object.keys(constantsData.fields).map(eachField => {
          if (this.categories.indexOf(constantsData.fields[eachField].category) === -1) {
            this.categories.push(constantsData.fields[eachField].category);
          }
          if (constantsData.fields[eachField].calculate) {
            this.$watch(eachField, function(newValue, oldValue) {
              if (vm.initialized) {
                this.changes.push({
                  field: eachField,
                  change: newValue - oldValue
                });
              }
            });
          } else {
            this.$set(this.constants, eachField, constantsData.fields[eachField].defaultValue);
            this.$watch('constants.'+eachField, function(newValue, oldValue) {
              if (vm.initialized) {
                this.changes.push({
                  field: eachField,
                  change: newValue - oldValue
                });
              }
            });
          }
        });
        this.$nextTick(() => {
          this.initialized = true;
        })
      },

      getCategoryFields(category) {
        let categoryFields = {};
        Object.keys(constantsData.fields).filter(eachField => {
          return constantsData.fields[eachField].category === category;
        }).forEach(eachField => {
          categoryFields[eachField] = constantsData.fields[eachField];
        });
        return categoryFields;
      },

      getConstantValue(key) {
        let field = constantsData.fields[key];
        if (field.calculate) {
          return Number.isInteger(this[key]*1) ? this[key] : (this[key]*1).toFixed(2);
        } else {
          return Number.isInteger(this.constants[key]*1) ? this.constants[key] : (this.constants[key]*1).toFixed(2);
        }
      },

      updateValue(changedValue, key) {
        this.changes = [];
        this.constants[key] = changedValue;
      },

      getChanges(key) {
        return this.changes.find(eachChange => eachChange.field === key);
      }
    },
  }
</script>