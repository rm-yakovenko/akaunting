<template>
  <div :id="'search-field-' + _uid" class="searh-field tags-input__wrapper">
    <div class="tags-group" v-for="(filter, index) in filtered" :index="index">
      <span v-if="filter.option" class="el-tag el-tag--primary el-tag--small el-tag--light el-tag-option">
        {{ filter.option }}
      </span>

      <span v-if="filter.operator" class="el-tag el-tag--primary el-tag--small el-tag--light el-tag-operator">
        {{ (filter.operator == '=') ? operatorIsText : operatorIsNotText }}
      </span>

      <span v-if="filter.value" class="el-tag el-tag--primary el-tag--small el-tag--light el-tag-value">
        {{ filter.value }}
        <i class="el-tag__close el-icon-close" @click="onFilterDelete(index)"></i>
      </span>
    </div>

    <div class="dropdown input-full">
      <input
        v-if="!show_date"
        type="text"
        class="form-control"
        :placeholder="placeholder"
        :ref="'input-search-field-' + _uid"
        v-model="search"
        @focus="onInputFocus"
        @input="onInput"
        @keyup.enter="onInputConfirm"
      />

      <flat-picker
        v-else
        @on-open="onInputFocus"
        :config="dateConfig"
        class="form-control datepicker"
        :placeholder="placeholder"
        :ref="'input-search-date-field-' + _uid"
        v-model="search"
        @focus="onInputFocus"
        @input="onInputDateSelected"
        @keyup.enter="onInputConfirm"
      >
      </flat-picker>

      <button type="button" class="btn btn-link clear" @click="onSearchAndFilterClear">
        <i class="el-tag__close el-icon-close"></i>
      </button>

      <div :id="'search-field-option-' + _uid" class="dropdown-menu" :class="[{'show': visible.options}]">
        <li ref="" class="dropdown-item" v-for="option in filteredOptions" :data-value="option.key">
          <button type="button" class="btn btn-link" @click="onOptionSelected(option.key)">{{ option.value }}</button>
        </li>
        <li ref="" v-if="search" class="dropdown-item">
          <button type="button" class="btn btn-link" @click="onInputConfirm">{{ searchText }}</button>
        </li>
      </div>

      <div :id="'search-field-operator-' + _uid" class="dropdown-menu" :class="[{'show': visible.operator}]">
        <li ref="" class="dropdown-item">
          <button type="button" class="btn btn-link" @click="onOperatorSelected('=')">{{ operatorIsText }}<span class="btn-helptext d-none">is</span></button>
        </li>
        <li ref="" class="dropdown-item">
          <button type="button" class="btn btn-link" @click="onOperatorSelected('!=')">{{ operatorIsNotText }}<span class="btn-helptext d-none">is not</span></button>
        </li>
      </div>

      <div :id="'search-field-value-' + _uid" class="dropdown-menu" :class="[{'show': visible.values}]">
        <li ref="" class="dropdown-item" v-for="(value) in filteredValues" :data-value="value.key">
          <button type="button" class="btn btn-link" @click="onValueSelected(value.key)">{{ value.value }}</button>
        </li>
        <li ref="" class="dropdown-item" v-if="!filteredValues.length">
          <button type="button" class="btn btn-link">{{ noMatchingDataText }}</button>
        </li>
      </div>
    </div>
  </div>
</template>

<script>
import flatPicker from "vue-flatpickr-component";
import "flatpickr/dist/flatpickr.css";

export default {
  name: 'akaunting-search',

  components: {
    flatPicker
  },

  props: {
    placeholder: {
      type: String,
      default: 'Search or filter results...',
      description: 'Input placeholder'
    },
    searchText: {
      type: String,
      default: 'Search for this text',
      description: 'Input placeholder'
    },
    operatorIsText: {
      type: String,
      default: 'is',
      description: 'Operator is "="'
    },
    operatorIsNotText: {
      type: String,
      default: 'is not',
      description: 'Operator is not "!="'
    },
    noDataText: {
        type: String,
        default: 'No Data',
        description: "Selectbox empty options message"
    },
    noMatchingDataText: {
        type: String,
        default: 'No Matchign Data',
        description: "Selectbox search option not found item message"
    },
    value: {
      type: String,
      default: null,
      description: 'Search attribute value'
    },
    filters: {
      type: Array,
      default: () => [],
      description: 'List of filters'
    },

    dateConfig: null,
  },

  model: {
    prop: 'value',
    event: 'change'
  },

  data() {
    return {
      filter_list: this.filters, // set filters prop assing it.
      search: '', // search cloumn model
      filtered:[], // Show selected filters
      filter_index: 0, // added filter count
      filter_last_step: 'options', // last fitler step
      visible: { // Which visible dropdown
        options: false,
        operator: false,
        values: false,
      },

      option_values: [],
      selected_options: [],
      selected_values: [],
      values: [],
      current_value: null,
      show_date: false,
    };
  },

  methods: {
    onInputFocus() {
      if (!this.filter_list.length) {
        return;
      }

      if (this.filter_last_step != 'values' || (this.filter_last_step == 'values' && this.selected_options[this.filter_index].type != 'date')) {
        this.visible[this.filter_last_step] = true;

        this.$nextTick(() => {
          this.$refs['input-search-field-' + this._uid].focus();
        });
      } else {
        this.show_date = true;

        this.$nextTick(() => {
          this.$refs['input-search-date-field-' + this._uid].focus();
        });
      }

      console.log('Focus :' + this.filter_last_step);
    },

    onInputDateSelected(event) {
      this.filtered[this.filter_index].value = event;

      this.selected_values.push({
        key: event,
        value: event,
      });

      this.$emit('change', this.filtered);

      this.show_date = false;
      this.search = '';

      this.$nextTick(() => {
        this.$refs['input-search-field-' + this._uid].focus();
      });

      this.filter_index++;

      if (this.filter_list.length) {
        this.visible = {
          options: true,
          operator: false,
          values: false,
        };
      } else {
        this.visible = {
          options: false,
          operator: false,
          values: false,
        };
      }

      this.show_date = false;

      this.filter_last_step = 'options';
    },

    onInput(evt) {
      this.search = evt.target.value;

      let option_url = this.selected_options[this.filter_index].url;

      if (this.search) {
        option_url += '?search="' + this.search + '" limit:10';
      }

      if (option_url) {
        window.axios.get(option_url)
        .then(response => {
            this.values = [];

            let data = response.data.data;

            data.forEach(function (item) {
              this.values.push({
                key: item.id,
                value: item.name
              });
            }, this);

            this.option_values[value] = this.values;
        })
        .catch(error => {

        });
      }

      this.$emit('input', evt.target.value);
    },

    onInputConfirm() {
      let path = window.location.href.replace(window.location.search, '');
      let args = '';

      if (this.search) {
        args += '?search="' + this.search + '" ';
      }

      let now = new Date();
      now.setTime(now.getTime() + 1 * 3600 * 1000);
      let expires = now.toUTCString();

      let search_string = {};
      search_string[path] = {};

      this.filtered.forEach(function (filter, index) {
        if (!args) {
          args += '?search=';
        }

        args += this.selected_options[index].key + ':' + this.selected_values[index].key + ' ';

        search_string[path][this.selected_options[index].key] = {
          'key': this.selected_values[index].key,
          'value': this.selected_values[index].value
        };
      }, this);

      Cookies.set('search-string', search_string, expires);

      window.location = path + args;
    },

    onOptionSelected(value) {
      this.current_value = value;

      let option = false;
      let option_url = false;

      for (let i = 0; i < this.filter_list.length; i++) {
        if (this.filter_list[i].key == value) {
          option = this.filter_list[i].value;

          if (typeof this.filter_list[i].url !== 'undefined' && this.filter_list[i].url) {
            option_url = this.filter_list[i].url;
          }

          if (typeof this.filter_list[i].type !== 'undefined' && this.filter_list[i].type == 'boolean') {
            this.option_values[value] = this.filter_list[i].values;
          }

          this.selected_options.push(this.filter_list[i]);
          this.filter_list.splice(i, 1);
          break;
        }
      }

      // Set filtered select option
      this.filtered.push({
        option: option,
        operator: false,
        value: false
      });

      this.$emit('change', this.filtered);

      this.search = '';

      if (!this.option_values[value] && option_url) {
        window.axios.get(option_url)
        .then(response => {
            let data = response.data.data;

            data.forEach(function (item) {
              this.values.push({
                key: (item.code) ? item.code : item.id,
                value: (item.title) ? item.title : (item.display_name) ? item.display_name : item.name
              });
            }, this);

            this.option_values[value] = this.values;
        })
        .catch(error => {

        });
      } else {
        this.values = (this.option_values[value]) ? this.option_values[value] : [];
      }

      this.$nextTick(() => {
        this.$refs['input-search-field-' + this._uid].focus();
      });

      this.visible = {
        options: false,
        operator: true,
        values: false,
      };

      this.filter_last_step = 'operator';
    },

    onOperatorSelected(value) {
      this.filtered[this.filter_index].operator = value;

      this.$emit('change', this.filtered);

      if (this.selected_options[this.filter_index].type != 'date') {
        this.$nextTick(() => {
          this.$refs['input-search-field-' + this._uid].focus();
        });

        this.visible = {
          options: false,
          operator: false,
          values: true,
        };
      } else {
        this.show_date = true;

        this.$nextTick(() => {
          this.$refs['input-search-date-field-' + this._uid].focus();
        });

        this.visible = {
          options: false,
          operator: false,
          values: false,
        };
      }

      this.filter_last_step = 'values';
    },

    onValueSelected(value) {
      let select_value = false;

      for (let i = 0; i < this.values.length; i++) {
        if (this.values[i].key == value) {
          select_value = this.values[i].value;

          this.selected_values.push(this.values[i]);
          this.option_values[this.current_value].splice(i, 1);
          break;
        }
      }

      this.filtered[this.filter_index].value = select_value;

      this.$emit('change', this.filtered);

      this.$nextTick(() => {
        this.$refs['input-search-field-' + this._uid].focus();
      });

      this.filter_index++;

      if (this.filter_list.length) {
        this.visible = {
          options: true,
          operator: false,
          values: false,
        };
      } else {
        this.visible = {
          options: false,
          operator: false,
          values: false,
        };
      }

      this.show_date = false;

      this.filter_last_step = 'options';
    },

    onFilterDelete(index) {
      this.filter_list.push(this.selected_options[index]);

      this.filter_index--;

      this.filtered.splice(index, 1);
      this.selected_options.splice(index, 1);
    },

    onSearchAndFilterClear() {
      this.filtered = [];
      this.search = '';

      Cookies.remove('search-string');

      this.onInputConfirm();
    },

    closeIfClickedOutside(event) {
      if (!document.getElementById('search-field-' + this._uid).contains(event.target) && event.target.className != 'btn btn-link') {
          this.visible.options = false;
          this.visible.operator = false;
          this.visible.values = false;

          document.removeEventListener('click', this.closeIfClickedOutside);
      }
    },
  },

  created() {
    let path = window.location.href.replace(window.location.search, '');

    let cookie = Cookies.get('search-string');

    if (cookie != undefined) {
      cookie = JSON.parse(cookie)[path];
    }

    if (this.value) {
      let search_string = this.value.split(' ');

      search_string.forEach(function (string) {
        if (string.search(':') === -1) {
          this.search = string;
        } else {
          let filter = string.split(':');
          let option = '';
          let value = '';

          this.filter_list.forEach(function (_filter, i) {
            if (_filter.key == filter[0]) {
              option = _filter.value;

              _filter.values.forEach(function (_value) {
                if (_value.key == filter[1]) {
                  value = _value.value;
                }
              }, this);

              if (!value && (cookie != undefined && cookie[_filter.key])) {
                value = cookie[_filter.key].value;
              }

              this.selected_options.push(this.filter_list[i]);
              this.filter_list.splice(i, 1);

              this.option_values[_filter.key] = _filter.values;

              _filter.values.forEach(function (value, j) {
                if (value.key == filter[1]) {
                  this.selected_values.push(value);

                  this.option_values[_filter.key].splice(j, 1);
                }
              }, this);

              if (cookie != undefined && cookie[_filter.key]) {
                  this.selected_values.push(cookie[_filter.key]);
              }
            }
          }, this);

          this.filtered.push({
            option: option,
            operator: '=',
            value: value
          });

          this.filter_index++;
        }
      }, this);
    }
  },

  computed: {
    filteredOptions() {
      this.filter_list.sort(function (a, b) {
          var nameA = a.value.toUpperCase(); // ignore upper and lowercase
          var nameB = b.value.toUpperCase(); // ignore upper and lowercase

          if (nameA < nameB) {
            return -1;
          }

          if (nameA > nameB) {
            return 1;
          }

          // names must be equal
          return 0;
      });

      return this.filter_list.filter(option => {
        return option.value.toLowerCase().includes(this.search.toLowerCase())
      });
    },

    filteredValues() {
      this.values.sort(function (a, b) {
          var nameA = a.value.toUpperCase(); // ignore upper and lowercase
          var nameB = b.value.toUpperCase(); // ignore upper and lowercase

          if (nameA < nameB) {
            return -1;
          }

          if (nameA > nameB) {
            return 1;
          }

          // names must be equal
          return 0;
      });

      return this.values.filter(value => {
        return value.value.toLowerCase().includes(this.search.toLowerCase())
      })
    }
  },

   watch: {
      visible: {
        handler: function(newValue) {
            if (newValue) {
                document.addEventListener('click', this.closeIfClickedOutside);
            }
        },
        deep: true
      }
    },
};
</script>

<style>
.searh-field {
    border: 1px solid #dee2e6;
    border-radius: 0.25rem;
}

.searh-field .tags-group {
  display: contents;
}

.searh-field .el-tag.el-tag--primary {
  background: #f6f9fc;
  background-color: #f6f9fc;
  border-color: #f6f9fc;
  color: #8898aa;
  margin-top: 7px;
}

.searh-field .tags-group:hover > span {
  background:#cbd4de;
  background-color: #cbd4de;
  border-color: #cbd4de;
}

.searh-field .el-tag.el-tag--primary .el-tag__close.el-icon-close {
  color: #8898aa;
}

.searh-field .el-tag-option {
  border-radius: 0.25rem 0 0 0.25rem;
  margin-left: 10px;
}

.searh-field .el-tag-operator {
  border-radius: 0;
  margin-left: -1px;
  margin-right: -1px;
}

.searh-field .el-tag-value {
  border-radius: 0 0.25rem 0.25rem 0;
  margin-right: 10px;
}

.searh-field .el-select.input-new-tag {
    width: 100%;
}

.searh-field .input-full {
  width: 100%;
}

.searh-field .btn.btn-link {
  width: inherit;
  text-align: left;
  display: flex;
  margin: 0;
  text-overflow: inherit;
  text-align: left;
  text-overflow: ellipsis;
  padding: unset;
  color: #212529;
}

.searh-field .btn.btn-link.clear {
  position: absolute;
  top: 0;
  right: 0;
  width: 45px;
  height: 45px;
  -webkit-box-pack: center;
  -ms-flex-pack: center;
  justify-content: center;
  -webkit-box-align: center;
  -ms-flex-align: center;
  align-items: center;
  color: #adb5bd;
  opacity: 1;
}

.searh-field .btn.btn-link.clear:hover {
  color: #8898aa;
}

.searh-field .btn-helptext {
  margin-left: auto;
  color: var(--gray);
}

.searh-field .btn:not(:disabled):not(.disabled):active:focus,
.searh-field .btn:not(:disabled):not(.disabled).active:focus {
  -webkit-box-shadow: none !important;
    box-shadow: none !important;
}

.searh-field .form-control.datepicker.flatpickr-input {
  padding: inherit !important;
}
</style>
