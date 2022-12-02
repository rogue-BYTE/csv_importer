<template>
  <div>
    <slot name="trigger">
      <b-button type="is-light" @click="openCSVImporter">
        Import
      </b-button>
    </slot>
  </div>
</template>

<script>
import CSVImporter from "./CSVImporter.vue";

export default {
  name: "CSVImportTrigger",
  components: {
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
  methods: {
    openCSVImporter() {
      this.$buefy.modal.open({
        parent: this.$root,
        component: CSVImporter,
        hasModalCard: true,
        fullScreen: true,
        canCancel: ['escape'],
        props: {
          schema: this.schema
        },
        trapFocus: true
      })
    }
  }
}

</script>
