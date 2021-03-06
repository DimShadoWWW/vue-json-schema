<script>
  import VxInput from '@vx-components/input'
  import VxSelect from '@vx-components/select'
  import VxTextarea from '@vx-components/textarea'
  import VxFileinput from '@vx-components/fileinput'
  import VxCheckbox from '@vx-components/checkbox'

  import { clone } from './lib/object'
  import { loadFields } from './lib/parser'

  const inputs = {
    select: VxSelect,
    checkbox: VxCheckbox,
    textarea: VxTextarea,
    input: VxInput,
    file: VxFileinput
  }
  const components = {}

  Object.keys(inputs).forEach((type) => {
    components[inputs[type].name] = inputs[type]
  })

  export default {
    name: 'form-schema',
    props: {
      /**
       * The JSON Schema object. Use the `v-if` directive to load asynchronous schema.
       */
      schema: { type: Object, required: true },

      /**
       * Use this directive to create two-way data bindings with the component. It automatically picks the correct way to update the element based on the input type.
       * @model
       */
      value: { type: Object, default: () => ({}) },

      /**
       * This property indicates whether the value of the control can be automatically completed by the browser. Possible values are: `off` and `on`.
       */
      autocomplete: { type: String },

      /**
       * This Boolean attribute indicates that the form is not to be validated when submitted.
       */
      novalidate: { type: Boolean },

      dataClassError: { type: String, default: 'uk-form-danger' }
    },
    data: () => ({
      default: {},
      fields: [],
      error: null
    }),
    created () {
      loadFields(this, clone(this.schema))
      this.default = clone(this.value)
    },
    render (createElement) {
      const nodes = []

      if (this.schema.title) {
        nodes.push(createElement('h1', this.schema.title))
      }

      if (this.error) {
        nodes.push(createElement('div', {
          class: this.dataClassError,
          attrs: { 'uk-alert': 'uk-alert' }
        }, [
          createElement('a', {
            class: 'uk-alert-close',
            on: {
              click: this.clearErrorMessage
            },
            attrs: { 'uk-close': 'uk-close' }
          }),
          createElement('p', this.title)
        ]))
      }

      if (this.fields.length) {
        const formNodes = []

        this.fields.forEach((field) => {
          if (!field.hasOwnProperty('data-class-error')) {
            field['data-class-error'] = this.dataClassError
          }

          delete field.value

          const component = inputs[field.type] || inputs.input
          const input = createElement(component.name, {
            ref: field.name,
            props: field,
            domProps: {
              value: this.value[field.name]
            },
            on: {
              input: () => {
                this.value[field.name] = event.target.value
              },
              change: this.changed
            },
          })

          const formControlsNodes = []

          if (field.label) {
            if (field.type !== 'checkbox' && field.type !== 'radio') {
              formControlsNodes.push(createElement('label', {
                class: 'uk-form-label',
                attrs: {
                  for: field.id
                }
              }, field.label))
            }
          }

          if (field.description) {
            formControlsNodes.push(createElement('br'))
            formControlsNodes.push(createElement('small', field.description))
          }

          switch (field.type) {
            case 'radio':
            case 'checkbox':
              const labelNode = createElement('label', {
                attrs: { title: field.title }
              }, [input])

              formControlsNodes.push(labelNode)
              break

            default:
              formControlsNodes.push(input)
          }

          const formControlsNode = createElement('div', {
            class: 'uk-form-controls'
          }, formControlsNodes)

          const marginNode = createElement('div', {
            class: 'uk-margin'
          }, [formControlsNode])

          formNodes.push(marginNode)
        })

        if (this.$slots.hasOwnProperty('default')) {
          formNodes.push(this.$slots.default)
        } else {
          formNodes.push(createElement('button', {
            class: 'uk-button uk-button-default',
            attrs: { type: 'submit' }
          }, 'Submit'))
        }

        nodes.push(createElement('form', {
          ref: '__form',
          class: 'uk-form-stacked',
          attrs: {
            autocomplete: this.autocomplete,
            novalidate: this.novalidate
          },
          on: {
            submit: (event) => {
              event.stopPropagation()
              this.submit(event)
            },
            invalid: this.invalid
          }
        }, formNodes))
      }

      return createElement('div', nodes)
    },
    mounted () {
      this.reset()
    },
    methods: {
      /**
       * @private
       */
      changed (e) {
        /**
         * Fired when an form input value is changed.
         */
        this.$emit('change', e)
      },

      /**
       * Get a form input component
       */
      input (name) {
        if (!this.$refs[name]) {
          throw new Error(`Undefined input reference '${name}'`)
        }
        return this.$refs[name][0]
      },

      /**
       * @private
       */
      invalid (e) {
        /**
         * Fired when a submittable element has been checked and doesn't satisfy its constraints. The validity of submittable elements is checked before submitting their owner form.
         */
        this.$emit('invalid', e)
      },

      /**
       * Reset the value of all elements of the parent form.
       */
      reset () {
        for (let key in this.default) {
          this.$set(this.value, key, this.default[key])
        }
      },

      /**
       * Send the content of the form to the server
       */
      submit () {
        if (this.$refs.__form.checkValidity()) {
          /**
           * Fired when a form is submitted
           */
          this.$emit('submit')
        }
      },

      /**
       * Set a message error.
       */
      setErrorMessage (message) {
        this.error = message
      },

      /**
       * clear the message error.
       */
      clearErrorMessage () {
        this.error = null
      }
    },
    components: components
  }
</script>
