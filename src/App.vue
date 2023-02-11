<template>
  <v-app>
    <v-main>
      <v-container>
        <v-row>
          <v-col cols="12" md="6" lg="6" sm="12" xs="12" offset-md="3">
            <h1 class="text-center text-h1 mt-2 mb-2">Task App</h1>
            <v-row>
              <v-col cols="9" md="10" sm="9">
                <v-text-field
                  v-model="input.task"
                  density="compact"
                  variant="solo"
                  :label="edit ? 'Edit task' : 'Add new task'"
                  single-line
                  hide-details
                  class="mb-3 mt-3"
                ></v-text-field>
              </v-col>
              <v-col  cols="3" md="2" sm="3">
                <v-btn v-show="!edit" @click="saveTask" :disabled="tasks.length >= 10" class="mt-3">Save</v-btn>
                <v-btn v-show="edit" @click="updateTask" class="mt-3">Update</v-btn>
              </v-col>
            </v-row>
            <p class="text-caption ml-2 mb-2 text-red" v-show="tasks.length >= 10">There can only be 10 tasks, delete some to add new tasks</p>
            <v-card
              elevation="2"
            >
            
            <v-list lines="two">
              <v-list-item
                v-for="task in tasks"
                :key="task.id"
                :title="task.task"
                :subtitle="dataFormat(task.created_at)"
              >
                <template v-slot:prepend>
                  <v-avatar :color="task.status == 'pending' ? 'amber' : 'success'">
                    <v-icon color="white">mdi-clipboard-text</v-icon>
                  </v-avatar>
                </template>

                <template v-slot:append>
                  <v-btn
                    color="blue-darken-1"
                    icon="mdi-pencil"
                    variant="text"
                    @click="editItem(task.id)"
                  ></v-btn>
                  <v-btn
                    color="red-darken-1"
                    icon="mdi-delete"
                    variant="text"
                    @click="deleteItem(task.id)"
                  ></v-btn>
                </template>
              </v-list-item>
            </v-list>
            </v-card>
          </v-col>
        </v-row>
      </v-container>
    </v-main>
    <v-card>
            <v-footer
            :padless="padless"
            >
            <v-card
                style="background-color: #ffffff;"
                flat
                tile
                width="100%"
                class="text-center"
            >
                <v-divider></v-divider>

                <v-card-text class="">
                {{ new Date().getFullYear() }} — <a href="https://www.omartoledo.com"><strong>Omar Toledo</strong></a> 
                </v-card-text>
            </v-card>
            </v-footer>
        </v-card>
    <v-snackbar
      v-model="snackbar"
      multi-line
      :timeout="timeout"
    >
      {{ text }}

      <template v-slot:actions>
        <v-btn
          color="red"
          variant="text"
          @click="snackbar = false"
        >
          Close
        </v-btn>
      </template>
    </v-snackbar>
  </v-app>
</template>

<script>
import axios from 'axios'
import moment from 'moment'
export default {
  name: 'App',

  data: () => ({
    padless: true,
    input: {
      id: null,
      task: null,
      status: null
    },
    tasks: [],
    edit: false,
    rules: {
          required: value => !!value || 'Required.',
          min: v => v.length >= 0 || 'Error',
        },
    snackbar: false,
    text: '',
    timeout: 2000,
  }),
  head() {
        return {
        title:
            "Task App | Omar Toledo",
        meta: [
            { charset: 'utf-8' },
            { name: 'viewport', content: 'width=device-width, initial-scale=1' },
            { name: 'keywords', content: 'Developer, Vue, JavaScript, API, Vuetify, Software, Code, AWS, Serverless, Lambda, API Gateway, DynamoDB, S3, CloudFront, Cloud Computing'},
            {
            hid: "description",
            name: "description",
            content:
                "Task App | Omar Toledo"
            }
        ]
        };
    },
  mounted () {
    this.getTasks()
  },  
  methods: {
    async saveTask() {
      try {
        if( this.input.task != '' ) {
          const response = await axios.post(`${process.env.VUE_APP_API_URL}/tasks?task=${this.input.task}`)

          if ( response.status == 200) {
            this.input.task = null
            this.getTasks()

            this.snackbar = true
            this.text = 'Task created successfully'
          } else {
            alert('Error')
          }
          
        }
      } catch (error) {
        console.log(error);
      }
    },
    async getTasks () {
      try {
        const response = await axios.get(`${process.env.VUE_APP_API_URL}/tasks`)

        const { data } = response

        this.tasks = data.Items

      } catch (error) {
        console.log(error);
      }
    },
    editItem (id) {
      const task = this.tasks.find(item => item.id == id)
      
      this.edit = true

      this.input.task = task.task
      this.input.id = task.id

    },
    async updateTask () {
      try {
        if( this.input.task != '' ) {
          const response = await axios.put(`${process.env.VUE_APP_API_URL}/tasks?task=${this.input.task}&id=${this.input.id}&status=pending`)

          if ( response.status == 200) {
            const index = this.tasks.findIndex(item => item.id == this.input.id)
            this.tasks[index].task = this.input.task

            this.edit = false

            this.input.task = null
            this.input.id = null

            this.snackbar = true
            this.text = 'Task updated successfully'
          } else {
            alert('Error')
          }
          
        }
      } catch (error) {
        console.log(error);
      }
    },
    async deleteItem (id) {
      try {
        
        const response = await axios.delete(`${process.env.VUE_APP_API_URL}/tasks?id=${id}`)

        if ( response.status == 200) {
          const index = this.tasks.findIndex(item => item.id == this.input.id)
          this.tasks.splice(index, 1)

          this.snackbar = true
          this.text = 'Task deleted successfully'

        } else {
          alert('Error')
        }
      } catch (error) {
        console.log(error);
      }
    },
    dataFormat (date) {
      return moment(date).fromNow()
    }
  }
}
</script>
