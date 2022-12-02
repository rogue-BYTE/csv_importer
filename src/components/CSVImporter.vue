<template>
  <div class="csv-importer modal-card">
    <header class="modal-card-head">
      <div class="modal-card-title is-flex is-vcenter">CSV Importer</div>
    </header>
    <section class="modal-card-body">
      <b-steps
        v-model="activeStep"
        :animated="false"
        :rounded="true"
        :has-navigation="false"
        :label-position="'bottom'"
      >
        <b-step-item step="1" label="Getting Started" :clickable="false">
          <div>
            <div>
              <div style="font-size: 20px; color: #7956D5">Get ready to import your data</div>
              <div>The following columns are supported. Some may be required, the rest are optional</div>
            </div>
            <vue-good-table
              style="width: 100%; margin-top: 1rem"
              :columns="header_detail_columns"
              :rows="schema.items"
            />
          </div>
        </b-step-item>

        <b-step-item step="2" label="Upload CSV" :clickable="false">
          <div
            class="step-container"
            style="border: 0.5px gray solid; border-radius: 0.5rem"
            @dragover.prevent
            @drop="drop"
            @dragleave.prevent
          >
            <div class="mb">Drag & drop files here <b>(or)</b></div>
            <label class="label-button" for="fileUploader">Browse Files</label
            >
            <input
              type="file"
              id="fileUploader"
              style="display: none"
              @change="onFileDrop"
              ref="fileUploadInput"
              accept=".csv"
            />
          </div>
        </b-step-item>

        <b-step-item step="3" label="Match Columns" :clickable="false">
          <div v-if="parsing">Parsing....</div>
          <div v-if="csv_file && parsed">
            <b-message 
              v-if="parsed"
              v-model="parsed"
              type="is-info"
              size="is-small"
            >
              Now match your columns by clicking the menu button in each column header.
            </b-message>
            <div
              v-for="str in error_str"
              :key="str"
              style="margin-bottom: 1rem"
            >
              <b-message 
                v-model="show_err_message"
                type="is-danger"
                size="is-small"
                aria-close-label="Close message"
              >
                {{ str }}
              </b-message>
            </div>
            <b-message 
              v-model="action_done"
              type="is-success"
              size="is-small"
            >
              CSV Imported Successfully, This Popup will close in 5 sec
            </b-message>
            <div class="mb" style="display: flex;">
              <template v-for="legend in Object.keys(schema.legends)">
                <div :key="legend" style="display: flex; align-items: center; margin-right: 50px;">
                  <div
                    :style="{
                      'background': schema.legends[legend].bg_color
                    }"
                    style="
                      height: 10px;
                      width: 80px;
                      margin-right: 10px;
                    "
                  ></div>
                  <p style="font-size: 12px">{{ schema.legends[legend].label }}</p>
                </div>
              </template>
            </div>
            <hot-table :settings="hotSettings" ref="hotTableComponent" style="margin-top: 1rem">
              <!-- <template v-for="header in csv_headers">
                <hot-column :key="header.data" :title="header.title" :data="header.data">
                </hot-column>
              </template> -->
            </hot-table>
          </div>
        </b-step-item>
      </b-steps>
    </section>
    <footer
      class="modal-card-foot"
    >
      <div style="display: flex; justify-content: space-between; width: 100%">
        <b-button
          style="margin-left: 50px"
          outlined
          type="is-secondary"
          :disabled="is_prev_disabled"
          @click.prevent="goPrev"
        >
          Previous
        </b-button>
        <b-button
          style="margin-right: 50px"
          filled
          type="is-primary"
          :disabled="is_next_disabled"
          @click.prevent="goNext"
        >
          {{ activeStep === 2 ? 'Finish': 'Next' }}
        </b-button>
      </div>
    </footer>
  </div>
</template>

<script>
/*
  Takes schema as props
  schema: {
    legends: {
      <validation method name>: {
        bg_color,
        text_color,
        label: <To be shown in the legends>
      }
    },
    items: [
      {
        field: <Column Name: is mandatory> TYPE(String),
        type: <numeric, date, email: is mandatory>TYPE(String),
        description: <Column display name: optional>TYPE(String),
        example: <Example of the data: optional>TYPE(String),
        validation: {
          <method: [mandatory, unique]>: {
            value: <value that the method will be evaluated with>
          }
        }TYPE(Object)
        canCreate: <true/false>
      }
    ]
  }
*/
import Papa from "papaparse";
import { HotTable } from "@handsontable/vue";
import { registerAllModules } from "handsontable/registry";
import {
  registerCellType, // cell types' registering function
  NumericCellType,
  TextCellType,
  DateCellType,
} from "handsontable/cellTypes";
import "handsontable/dist/handsontable.full.css";
import "vue-good-table/dist/vue-good-table.css";
import { VueGoodTable } from "vue-good-table";
// register Handsontable's modules
registerAllModules();
registerCellType(NumericCellType);
registerCellType(TextCellType);
registerCellType(DateCellType);

export default {
  name: "CSVImporter",
  components: {
    HotTable,
    VueGoodTable,
  },
  props: {
    schema: {
      type: Object,
      default: () => {
        return {
          legends: {
            mandatory: {
              bg_color: "orange",
              text_color: "black",
              label: "Is empty"
            },
            regex: {
              bg_color: "red",
              text_color: "white",
              label: "Invalid Value"
            },
            unique: {
              bg_color: "yellow",
              text_color: "black",
              label: "Duplicate"
            },
            unavailable: {
              bg_color: "gray",
              text_color: "black",
              label: "Nonexistent(Will be created)"
            }
          },
          items: [
            {
              field: "first_name",
              type: "text",
              validation: {
                mandatory: {
                  value: true,
                },
                unique: {
                  value: true
                }
              },
              description: "First Name",
              example: "John",
            },
            {
              field: "last_name",
              type: "text",
              validation: {
                mandatory: {
                  value: true,
                }
              },
              description: "Last Name",
              example: "Doe",
            },
            {
              field: "age",
              type: "numeric",
              validation: {
                mandatory: {
                  value: true,
                }
              },
              description: "Age",
              example: "42",
            },
            {
              field: "designation",
              type: "text",
              validation: {
                mandatory: {
                  value: true,
                }
              },
              description: "Designation",
              example: "Technical Manager",
            },
            {
              field: "department",
              type: "text",
              validation: {
                mandatory: {
                  value: true,
                }
              },
              description: "Department",
              items: ["Frontend", "Backend", "DevOps"],
              canCreate: true,
              createHooks: (x) => {
                console.log("creating ", x);
              },
              example: "Frontend",
            },
            {
              field: "qualification",
              type: "text",
              description: "Qualification",
              example: "MS",
            },
            {
              field: "date_of_joining",
              type: "date",
              validation: {
                mandatory: {
                  value: true,
                }
              },
              description: "Date of Joining",
              example: "2022-01-13",
            },
            {
              field: "email",
              type: "email",
              description: "Email",
              validation: {
                mandatory: {
                  value: true,
                },
                unique: {
                  value: true
                }
              },
              example: "john.doe@hotmail.com",
            },
          ]
        }
      }
    },
  },

  data() {
    return {
      action_done: false,
      activeStep: 0,
      show_mapping_tool: null,
      duplicateElements: {},
      validation_status: {},
      error_str: [],
      hotSettings: {
        data: [],
        height: "400px",
        columns: [],
        contextMenu: true,
        stretchH: "all",
        licenseKey: "non-commercial-and-evaluation",
        rowHeaders: true,
        colHeaders: (col) => {
          if (Object.keys(this.mapping).includes(this.csv_headers[col].data)) {
            return  `<b>${this.csv_headers[col].data} -> ${this.mapping[this.csv_headers[col].data]}</b>`;
          } else {
            return  `<b>${this.csv_headers[col].data}</b>`;
          }
        },
        dropdownMenu: {
          callback: (key, selection) => {
            const hot = this.$refs.hotTableComponent.hotInstance;
            const format = this.schema.items.find((f) => f.field === key);

            const col_idx = selection[0].start.col;
            console.log(col_idx);
            this.duplicateElements[col_idx] = [];
            this.mapping[this.csv_headers[col_idx].data] = key;
            if (format.canCreate) {
              this.data_to_be_created[format.field] = new Set();
            }
            this.hotSettings.columns[col_idx] = {
              data: this.hotSettings.columns[col_idx].data,
              mapped: true,
              ...(format?.validation && {
                renderer: (instance, td, row, col, prop, value, cellProperties) => {
                  // -----------------To remove lint start------------------
                  let x = false;
                  x && console.log(instance, td, row, col, prop, value, cellProperties);
                  // -----------------To remove lint end--------------------
                  if (col === col_idx) {
                    if (format.type === 'date') {
                      cellProperties.type = "date";
                      cellProperties.dateFormat = "YYYY-MM-DD";
                    }
                    if (format.validation?.unique) {
                      const rows = hot.getDataAtCol(col_idx);
                      this.duplicateElements[col_idx] = this.getDuplicates(rows);
                    }

                    // Validate the value
                    const { validation_success, failed_method } = this.validateCell(value, format, col_idx);
                    this.validation_status[`${row}-${col}`] = validation_success;
                    // Set styling for invalid (custom/predefined)
                    if (validation_success) {
                      if (format?.items?.includes(value) === false) {
                        this.data_to_be_created[format.field].add(value);
                        td.style.background = this.schema.legends.unavailable.bg_color;
                        td.style.color = this.schema.legends.unavailable.text_color;
                      }
                    }
                    else {
                      if (this.schema.legends[failed_method]) {
                        td.style.background = this.schema.legends[failed_method].bg_color;
                        td.style.color = this.schema.legends[failed_method].text_color;
                      } else {
                        td.style.background = "red";
                        td.style.color = "white";
                      }
                    }
                    td.innerHTML = value;
                    this.error_str = [];
                  }
                }
              }),
            };
            hot.updateSettings({
              columns: this.hotSettings.columns,
            });
            hot.validateCells();
            console.log(this.hotSettings.columns);
          },
          items: this.schema.items.map((x) => {
            return {
              key: x.field,
              name: x.field
            };
          }),
        },
      },
      csv_headers: null,

      csv_file: null,
      parsed: null,
      content: null,
      parsing: null,

      mapping: {},
      data_to_be_created: {},
      date_config: {
        type: "date",
        dateFormat: "YYYY-MM-DD",
        correctFormat: false,
        datePickerConfig: {
          // First day of the week (0: Sunday, 1: Monday, etc)
          firstDay: 0,
          showWeekNumber: true,
          numberOfMonths: 1,
          disableDayFn(date) {
            // Disable Sunday and Saturday
            return date.getDay() === 0 || date.getDay() === 6;
          },
        },
      },
      header_detail_columns: [
        {
          label: "COLUMN NAME",
          field: "field",
          sortable: false,
        },
        {
          label: "FORMAT",
          field: "type",
          sortable: false,
        },
        {
          label: "EXAMPLE",
          field: "example",
          sortable: false,
        },
        {
          label: "REQUIRED",
          field: "validation.mandatory.value",
          sortable: false,
        },
        {
          label: "DESCRIPTION",
          field: "description",
          sortable: false,
        },
      ],
      // Regex collection for type validation
      default_validators: {
        numeric: /^[0-9]+$/,
        email: /^[\w-.]+@([\w-]+\.)+[\w-]{2,4}$/
      }
    };
  },
  mounted() {},
  watch: {
    "hotSettings.data": {
      handler(n, o) {
        console.log(n, o);
      },
      deep: true,
    },
  },
  computed: {
    is_prev_disabled() {
      return !this.activeStep;
    },
    is_next_disabled() {
      if (this.activeStep === 1) {
        return !(this.csv_file && this.parsed);
      }
      if (this.activeStep === 2) {
        console.log('step 2');
      }
      return false;
    },
    show_err_message() {
      return this.error_str.length > 0;
    }
  },
  methods: {
    getRandomColor() {
      var letters = '0123456789ABCDEF';
      var color = '#';
      for (var i = 0; i < 6; i++) {
        color += letters[Math.floor(Math.random() * 16)];
      }
      return color;
    },
    getDuplicates(arr) {
      const uniqueElements = new Set(arr);
      const filteredElements = arr.filter(item => {
          if (uniqueElements.has(item)) {
              uniqueElements.delete(item);
          } else {
              return item;
          }
      });
      return [...new Set(filteredElements)]
    },
    validateCell(value, format, col_idx) {
      let validation_success = true;
      let failed_method = null;

      if (Object.keys(this.default_validators).includes(format.type)) {
        const reg = this.default_validators[format.type];
        if(!reg.test(value)) {
          failed_method = "regex";
          validation_success = false;
          return { validation_success, failed_method };
        }
      }
      for (const method in format.validation) {
        if (method === "mandatory") {
          if (!value && format.validation[method].value) {
            failed_method = method;
            validation_success = false;
            break;
          }
        } else if (method === "unique") {
          if (this.duplicateElements[col_idx].includes(value)) {
            failed_method = method;
            validation_success = false;
            break;
          }
        }
      }
      return { validation_success, failed_method };
    },
    goPrev() {
      if (this.activeStep === 2) {
        this.csv_file = null;
        this.hotSettings.data = [];
        this.hotSettings.columns = [];
        this.parsed = null;
        this.content = null;
        this.mapping = {};
        this.csv_headers = [];
        this.error_str = [];
        this.duplicateElements = {};
        this.validation_status =  {};
        this.data_to_be_created = {};
        this.$refs.hotTableComponent.hotInstance.destroy();
      }
      this.activeStep = this.activeStep - 1;
    },
    async goNext() {
      if (this.activeStep === 2) {
        this.error_str = [];
        // Check if every mandatory is mapped
        const mapped_columns = Object.values(this.mapping);
        const unmappped_columns = this.schema.items.filter(item => {
          if (item.validation?.mandatory?.value) {
            if (!mapped_columns.includes(item.field)) {
              return true;
            }
          }
          return false;
        }).map(col => col.field);
        if (unmappped_columns.length) {
          this.error_str.push(`Columns ${unmappped_columns.join(', ')} are mandatory`);
        }
        // Check if every cell is validated
        if (Object.values(this.validation_status).includes(false)) {
          this.error_str.push("Some cells contain empty/invalid/duplicate values");
        }

        if (this.error_str.length) {
          console.log(this.error_str);
          return;
        } else {
          await this.postHook();
          this.action_done = true;
          await new Promise(resolve => setTimeout(resolve, 5000));
          this.$emit("close");
        }
      } else {
        this.activeStep = this.activeStep + 1;
      }
    },
    async postHook() {
      for (const col in this.data_to_be_created) {
        const format = this.schema.items.find(item => item.field === col);
        await format.createHooks(this.data_to_be_created[col]);
      }
    },
    drop(e) {
      e.preventDefault();
      this.$refs.fileUploadInput.files = e.dataTransfer.files;
      this.onFileDrop();
    },
    onFileDrop() {
      this.csv_file = this.$refs.fileUploadInput.files[0];
      this.parsing = true;
      Papa.parse(this.csv_file, {
        header: true,
        skipEmptyLines: true,
        complete: function (results) {
          this.goNext();
          this.content = results.data;
          this.csv_headers = Object.keys(this.content[0]).map((header) => {
            return {
              title: header,
              data: header,
            };
          });

          // Table attributes update
          this.hotSettings.data = this.content;
          this.hotSettings.columns = this.csv_headers.map((header) => {
            return {
              data: header.data,
            };
          });
          // this.hotSettings.colHeaders = this.csv_headers.map(
          //   (header) => header.data
          // );

          this.parsed = true;
          this.parsing = false;
        }.bind(this),
      });
    },
  },
};
</script>

<style scoped>
.label-button {
  padding: 1%;
  color: white;
  background: #7956D5;
  border-radius: 0.5rem;
}
.mb {
  margin-bottom: 1rem;
}
.step-container {
  display: flex;
  justify-content: center;
  align-items: center;
  flex-direction: column;
  width: 100%;
  height: 450px;
}
.csv-importer {
}
.dropdown-wrapper {
  position: relative;
}
.mapping-dropdown {
  position: absolute;
  background-color: #fff;
  overflow: visible;
  z-index: 10000;
  min-width: 300px;
  box-shadow: 5px 10px 18px #888888;
  background: white;
  -webkit-box-sizing: border-box;
  -moz-box-sizing: border-box;
  box-sizing: border-box;
  max-height: 50vh;
  overflow-y: scroll;
  padding: 1%;
}
.dropdown-option {
  padding: 1.5% 1% 1.5% 1%;
  border-bottom: 0.1px #d9d9d9 solid;
}
</style>
