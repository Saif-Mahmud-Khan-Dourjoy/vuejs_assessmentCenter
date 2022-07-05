<template>
  <div class="card">
    <div class="card-body">
      <h5 class="card-title">New Question Create</h5>
      <hr />
      <div class="row">
        <div class="col-md-6">
          <label>Question Type</label>
          <select
            class="form-select"
            aria-label="Default select example"
            placeholder="select Type"
            ref="question_type"
            @change="type()"
          >
            <option selected>Select One Type</option>
            <option value="checkbox">MCQ (Multiple Answer)</option>
            <option value="radio">MCQ (Single Answer)</option>
            <option value="truefalse">True or False</option>
          </select>
        </div>
        <div class="col-md-6">
          <label>Question Tag</label> <br />
          <!-- <select
            multiple
            class="form-select qus_tag"
            aria-label="Default select example"
            v-model="question_tag"
          >
            <option :value="item.id" v-for="item in category" :key="item.id">
              {{ item.name }}
            </option>
          </select> -->
          <!-- <div
            class="my-multiselect"
            @click="tagShow"
            @blur="focused = false"
            tabindex="-1"
          >
            <div class="my-multiselect_options" v-show="focused">
              <div
                class="my-multiselect_option"
                :class="{
                  'my-multiselect_option--checked':
                    selected_category_name.checked,
                }"
                v-for="(item, index) in category"
                :key="item.id"
              >
                <div class="my-multiselect_checkbox"></div>
                <div class="my-multiselect_text px-5">{{ item.name }}</div>
              </div>
            </div>
          </div> -->
        </div>
      </div>

      <div class="row mt-2">
        <label>Question</label>

        <div class="p-3">
          <vue-editor
            id="question"
            ref="question"
            useCustomImageHandler
            @image-added="handleImageAdded"
            v-model="question"
          >
          </vue-editor>
        </div>
      </div>

      <div>
        <p>
          <span v-if="!true_false">Add options and </span> Tick the correct
          option
        </p>
      </div>

      <div v-if="multiple_ans" id="options_for_multiple_ans">
        <div v-for="(item, index) in mul_obj" :key="index + 1">
          <div class="row mt-2">
            <div class="col-1">{{ item.no }}.</div>
            <div class="col-1">
              <input
                type="checkbox"
                :id="'option-' + item.no"
                :value="item.no"
                :ref="'option_' + item.no"
                v-model="checkbox_num"
              />
            </div>
            <div class="col-10">
              <vue-editor
                v-bind:id="'option_text-' + item.no"
                :ref="'option_text_' + item.no"
                useCustomImageHandler
                @image-added="handleImageAdded"
                v-model="option_text[index]"
              >
              </vue-editor>
            </div>
          </div>
        </div>
      </div>

      <div v-if="single_ans" id="options_for_single_ans">
        <div v-for="(item, index) in mul_obj" :key="index + 1">
          <div class="row mt-2">
            <div class="col-1">{{ item.no }}.</div>
            <div class="col-1">
              <input
                type="radio"
                :id="'option-' + item.no"
                :value="item.no"
                :ref="'option-' + item.no"
                name="single_ans"
                v-model="radio_num"
              />
            </div>
            <div class="col-10">
              <vue-editor
                :id="'option_text-' + item.no"
                :ref="'option_text_' + item.no"
                useCustomImageHandler
                @image-added="handleImageAdded"
                v-model="option_text[index]"
              >
              </vue-editor>
            </div>
          </div>
        </div>
      </div>

      <div v-if="true_false" id="options_for_true_false_ans">
        <div class="row mt-2">
          <div class="col-1">1</div>
          <div class="col-1">
            <input
              type="radio"
              id="option-1"
              value="true"
              ref="option_1"
              name="true_false_ans"
              v-model="true_false_option"
            />
          </div>
          <div class="col-10">
            <input type="text" class="form-control" value="True" disabled />
          </div>
        </div>
        <div class="row mt-2">
          <div class="col-1">2.</div>
          <div class="col-1">
            <input
              type="radio"
              id="option-2"
              value="false"
              ref="option-2"
              name="true_false_ans"
              v-model="true_false_option"
            />
          </div>
          <div class="col-10">
            <input type="text" class="form-control" value="False" disabled />
          </div>
        </div>
      </div>

      <div class="mt-3 text-end" v-if="!true_false">
        <button @click="delOption()" class="btn btn-outline-danger del">
          Delete
        </button>
        <button @click="addNew()" class="btn btn-outline-info">Add More</button>
      </div>
      <!-- {{ formatedTag }} -->
      <div class="mt-3 text-center">
        <button class="btn btn-outline-success" @click="addQuestion()">
          Submit Qustion
        </button>
      </div>

      <div>
        <Multiselect
          mode="multiple"
          v-model="value"
          :close-on-select="true "
          :searchable="true"
          :create-option="true"
          :options="category_name"
          placeholder="Select Tag"
        />
      </div>
      {{ category_name }}
    </div>
  </div>
</template>

<script>
import axios from 'axios'
// import $ from 'jquery'
import { base_url } from '../../../Config.js'
import { VueEditor } from 'vue3-editor'
import Multiselect from '@vueform/multiselect'

export default {
  name: 'ButtonGroups',
  components: {
    VueEditor,
    Multiselect,
  },
  // props: {
  //   options: {
  //     type: Array,
  //     default: () => [],
  //   },
  //   value: {
  //     default: () => [],
  //   },
  //   placeholder: {
  //     type: String,
  //     default: 'Click To Select Tag',
  //   },
  //   displayProperty: {
  //     type: String,
  //     default: 'name',
  //   },
  //   valueProperty: {
  //     type: String,
  //     default: 'value',
  //   },
  // },

  data() {
    return {
      token: '',
      value: null,
      options: ['Batman', 'Robin', 'Joker'],
      checkbox_num: [],
      radio_num: null,
      true_false_option: null,
      option_text: [null],
      category: [],
      category_name: [],
      selected_category: [],
      selected_category_name: [],
      option_no: 1,
      question: '',
      question_type: '',
      question_tag: [],
      multiple_ans: true,
      single_ans: false,
      true_false: false,
      mul_obj: [
        {
          no: 1,
          checkbox_radio: 1,
          com: 1,
          model: 'option_text_1',
        },
      ],
      serial_no: '',
      focused: false,
    }
  },
  computed: {
    formatedTag() {
      let tag = this.category.map((op) => {
        let checked = this.selected_category_name.some((v) => v == op.name)
        return { ...op, checked }
      })
      return tag
    },
  },

  Created() {
    this.login()
    //this.getCategory()
  },
  beforeMount() {
    this.getCategory()
  },

  methods: {
    async login() {
      let data = {
        username: 'Admin',
        password: '123456789',
      }

      await axios
        .post(`http://localhost:8002/api/v1/login`, data)
        .then((response) => {
          let token = response.data.token
          this.token = token
          console.log(token)
          let userInfo = response.data.user
          localStorage.setItem('auth-token', token)
          localStorage.setItem('user-info', JSON.stringify(userInfo))
        })
    },
    async getCategory() {
      //let token = localStorage.getItem('auth-token')
      let token =
        'eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiJ9.eyJhdWQiOiIzIiwianRpIjoiMDQ5Y2E2Mjg5YzQxZjM5ZDlmZDI4ZWRkZDc1NmE2MjFkMzYzMTk2YjEyZWRjNzZhY2NkY2Q4MDg2YjYzZDQyZGNhNDUzMzdkZTMwYjYyODgiLCJpYXQiOjE2NTY5MDY5MjQuNzkwMjc0LCJuYmYiOjE2NTY5MDY5MjQuNzkwMjc4LCJleHAiOjE2ODg0NDI5MjQuNzIxMDczLCJzdWIiOiIxIiwic2NvcGVzIjpbXX0.qLYY01g9cQlMmxvgXpZT8yfHYbclMUHUYAFwrUUSlKL6z8k1FwSAgXqLqPQxOaWSSwVFGqbvGEh_PD-sxAebGkgpKtEmHATR-HfRvba2mu6KvU9PTH-uIZvjHBnA6HEkUI19GZFPPgejTPVx5OriQasq0un32vcjHqtYDVwUq2Hmp1bQEK1tB57caoKxIHHpZgfdXpdLDwsGtfyhIH6fCRG5_QuE03qqONkJ8bVRSqS8pXR62zL6nJ5AudVN4op0GQVbu60Cs2o1NeiSX-pGu0i3Dkdh0WWOmd_zxtBYgAnPyA9jxuX4HytzK1v5hoiAKeXNAMbdD9pWFT9C5u8gC3ycV4mG5DforsbMWh3qcQM34F4jeA0CRlr5WAIYbVRvZRL2MjI96XqKlDRU1aoPur9bhscTAB-Ws8rlSqFgyZb62i_PeoSLUG-2BwTI-y5yqrZQW0aLDD3Ku5jfNmGSjDQFELIg4WjchIcN7mk4aXSoDnCMdRB6cylYBTNrxuckKHz_XO2cdFkFl6W7_vm6kTpPrnc0bJgw3UjpjUEyMfwI7Zeiod9SZmMAY8cagJaOpR3ZHrY5oYIZM6njfD_r4QPiuumuLBw5yyzHiDiWLW-6lTg9vIV77jeWCCs_FppNDVANX1WbTQuRszKVqj-FwM3UlVwPGuUr8p_ki7zXH0g'
      var BASE_URL = 'http://localhost:8002/api/v1'
      var ENDPOINT = 'question-categories'
      await axios
        .create({
          baseURL: BASE_URL,
          headers: {
            Accept: 'application/json',
            Authorization: 'Bearer ' + token,
          },
        })
        .get(ENDPOINT)
        .then((res) => {
          console.log(res)
          this.category = res.data.question_categories
          for (let i = 0; i < this.category.length; i++) {
            this.category_name.push(this.category[i].name)
          }
        })
    },
    tagShow() {
      this.focused = !this.focused
    },
    // formatedTag() {
    //   let tag = this.category.map((option) => {
    //     return { ...option, checked: false }
    //   })
    // },
    addNew() {
      let no = this.option_no + 1
      this.option_no = no
      let obj = {
        no: this.option_no,
        checkbox_radio: this.option_no,
        com: this.option_no,
        model: 'option_text_' + this.option_no,
      }
      this.mul_obj.push(obj)
      this.option_text.push(null)
    },
    delOption() {
      if (this.mul_obj.length > 1) {
        this.mul_obj.pop()
        let no = this.option_no - 1
        this.option_no = no
        console.log(this.option_no)
      }
    },
    type() {
      this.question_type = this.$refs.question_type.value
      if (this.$refs.question_type.value == 'checkbox') {
        this.multiple_ans = true
        this.single_ans = false
        this.true_false = false
      } else if (this.$refs.question_type.value == 'radio') {
        this.multiple_ans = false
        this.single_ans = true
        this.true_false = false
      } else {
        this.multiple_ans = false
        this.single_ans = false
        this.true_false = true
      }
    },
    addQuestion() {
      let question_type = this.question_type

      if (question_type == 'checkbox') {
        let serial_no = ''
        for (let i = 1; i <= this.mul_obj.length; i++) {
          if (i == 1) {
            serial_no = serial_no + '' + i
          } else {
            serial_no = serial_no + '|' + i
          }
        }

        console.log(serial_no)
        let data = {
          question_type: this.question_type,
          question_text: this.question_type,
          option_type: this.question_type,
          profile_id: 1,
          category_id: 0,
          privacy: 0,
          publish_status: 0,
          no_of_option: this.option_no,
          no_of_used: 0,
          no_of_comments: 0,
          average_rating: 0,
          no_of_answer: this.checkbox_num.length,
          active: 0,
          serial_no: serial_no,
          option: this.option_text.join('|'),
          description: this.question,
          img: 'img',
          answer: this.checkbox_num.join(','),
          reference: 0,
          tags: this.question_tag.join(','),
        }
      }
    },

    handleImageAdded: function (file, Editor, cursorLocation, resetUploader) {
      var formData = new FormData()
      formData.append('image', file)

      axios({
        url: `${base_url}/upload`,
        method: 'POST',
        data: formData,
      })
        .then((result) => {
          console.log(result)
          const base_url = result.data.base_url // Get url from response
          const image_url = result.data.img_url
          const url = base_url + '/' + image_url
          Editor.insertEmbed(cursorLocation, 'image', url)
          resetUploader()
        })
        .catch((err) => {
          console.log(err)
        })
    },
  },
}
</script>
<style src="@vueform/multiselect/themes/default.css"></style>