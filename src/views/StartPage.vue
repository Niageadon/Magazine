<template xmlns:v-slot="http://www.w3.org/1999/XSL/Transform">
  <div>

    <div>
      <v-dialog>
      <quillEditor v-model="editedRecord.enable" class="mx-2 mb-5"  newRecord="false" :editRecord="editedRecord">
      </quillEditor>
      </v-dialog>
    </div>
    <!--:style="{ background: `rgb(${red}, ${green}, ${blue})` }"-->

    <v-container mt-4 fluid>
      <v-layout wrap xs12 justify-center>
        <v-flex xs12 md7>
          <v-card style="background-color: rgba(49,128,114,0.17)" class="elevation-13">
            <v-card-title>
              <v-layout wrap>
                <v-flex  xs6 md4 >
                  <v-select
                      v-model="newNote.recordType"
                      :items="getTypesArray"
                      item-text="state"
                      item-value="abbr"
                      label="Select"
                      persistent-hint
                      single-line
                  ></v-select>
                </v-flex> <!--note type-->
                <v-flex xs6 md4 class="offset-md1 offset-xs0">
                  <v-checkbox  append-icon="warning"  label="Important" v-model="newNote.isImportant"></v-checkbox>
                </v-flex> <!--"Important" checkbox-->
              </v-layout>
            </v-card-title>
            <v-divider/>

            <v-form v-model="validateRules.valid">
              <v-layout xs12 wrap column>
                <v-menu
                    ref="dateMenu"
                    v-model="newNote.dateMenu"
                    box
                    :close-on-content-click="false"
                    :nudge-right="50"
                    :return-value.sync="newNote.date"
                    lazy
                    transition="scale-transition"
                    offset-y
                    full-width
                    min-width="290px"
                >
                  <template v-slot:activator="{ on }">
                    <v-text-field
                        class="mx-2"
                        v-model="newNote.date"
                        label="Set note date"
                        prepend-inner-icon="date_range"
                        readonly
                        v-on="on"
                    ></v-text-field>
                  </template>
                  <v-date-picker  v-model="newNote.date" no-title scrollable>
                    <v-spacer></v-spacer>
                    <v-btn flat color="primary" @click="newNote.dateMenu = false">Cancel</v-btn>
                    <v-btn flat color="primary" @click="$refs.dateMenu.save(newNote.date)">Select</v-btn>
                  </v-date-picker>
                </v-menu><!--date-->




                  <quillEditor class="mx-2 mb-5" newRecord="true">
                  </quillEditor>


                <!--<v-textarea
                    v-on:keydown.tab="newNote.body = doTabulation(newNote.body,$event)"
                    box
                    name="input-7-4"
                    label="Main text"
                    v-model="newNote.body"
                    :rules="validateRules.bodyRule"
                ></v-textarea>--><!--body-->
              </v-layout>
            </v-form>

            <v-card-actions >
              <v-layout xs12 wrap >
                <v-flex  align-self-end  xs12 >
                  <v-btn
                      large
                      block
                      :disabled="!validateRules.valid"
                      @click="addNote"
                      color="primary">
                      Publish
                    <v-icon class="ml-2">cloud_download</v-icon>
                  </v-btn>
                </v-flex> <!--button-->
              </v-layout>
            </v-card-actions> <!--button-->
          </v-card>
        </v-flex>



        <v-flex mt-3 xs12 md7>
          <v-layout wrap class="noSelect" row xs12 justify-space-between>
            <v-flex md3 xs6 v-for="(type, i) in recordTypes" :key="i">
              <v-card
                  @click="selectedRecordTypeToShow = type.name"
                  style="text-align: center"
                  min-height="40"
                  :color="(selectedRecordTypeToShow === type.name)? type.color: 'gray'" hover>
                <v-card-text class="title">
                  {{type.name}}
                </v-card-text>
              </v-card>
            </v-flex>
          </v-layout>
        </v-flex>
      </v-layout>
    </v-container> <!--New note-->

    <v-container>
      <v-layout wrap justify-center>
        <v-flex mt-4 pb-4 xs12 md11 v-for="note in getRecord[selectedRecordTypeToShow]" :key="note.date">
          <v-stepper  v-model="note.step">

            <v-stepper-items >
              <v-stepper-content class="pa-0" step="1">
                <v-card class="elevation-10">
                  <v-card-title
                      class="headline lighten-2 font-weight-bold"
                      v-bind:class="{important: note.isImportant, usial: !note.isImportant}">
                    {{note.date}}
                  </v-card-title>
                <v-divider></v-divider>
                <v-card-text class="title">
                  <div v-html="note.body"></div>
                </v-card-text>
                <!--<v-responsive>
                  <v-img  src="https://cdn.vuetifyjs.com/images/carousel/sky.jpg"> </v-img>
                </v-responsive>-->
                <v-card-actions>

                  <v-btn
                      @click="editRecord(note); note.step = 2"
                      color="red lighten-2"
                      dark>
                    Редактировать запись
                  </v-btn>

                </v-card-actions>
              </v-card>
              </v-stepper-content>
              <!--content-->
              <v-stepper-content class="pa-0" step="2">
                <v-card class="elevation-24">
                  <v-card-title class="px-2">

                  </v-card-title>
                    <quillEditor v-model="editedRecord.enable" class="px-2"  newRecord="false" :editRecord="note">
                    </quillEditor>
                  <v-card-actions>
                    <v-btn class="mt-3" @click="saveEditChanges(note.i); note.step = 1" >Save changing</v-btn>
                  </v-card-actions>
                </v-card>
              </v-stepper-content>
              <!--edit-->
            </v-stepper-items>
          </v-stepper>
        </v-flex>
      </v-layout>
    </v-container> <!--Note's array-->
  </div>
</template>

<script>

  import quillEditor from "./main/Records/Editor"


  export default {
    name: "StartPage",

    components:{
      quillEditor
    },

    data(){
      return{

        quillData: 'bobi',
        //db: this.firebase.firestore(),
        recordOnEdition: false, //для v-dialog
        selectedRecordTypeToShow: 'note',

        recordTypes: [
          {name: 'note',      color: 'red'},
          {name: 'task',      color: 'blue'},
          {name: 'reminder',  color: 'yellow'},
          {name: 'med',       color: 'green'}

        ],

        validateRules:{
          valid: false,
          bodyRule: [
            v => !!v || 'Enter text',
          ]
        },

        newNote:{
          dateMenu: false,
          recordType: 'note',
          date: '',
          body: '',
          isImportant: false,
        },

        editedRecord:{
          recordType: '',
          date: '',
          body: '',
        },

      }
    },

    methods:{
      addNote() {
        //this.notes.push(this.newNote.note)
        let recordType = this.newNote.recordType.toLowerCase();

        let newRecord = {
          date:         this.newNote.date,
          isImportant:  this.newNote.isImportant,
          step: 1
          //body:         this.newNote.body,
        };
        this.$store.dispatch('NewRecord', {recordType, newRecord})
      },

      getCurrentDate(){
        // auto select current date
        let currentDate = new Date;
        let year = currentDate.getFullYear();
        let day = currentDate.getDate();
        day = (day.toString().length === 1)? '0' + day : day;
        let month = currentDate.getMonth() + 1;
        month = (month.toString().length === 1)? '0' + month : month;
        return(year + '-' + month + '-' + day )
      },

      doTabulation(text, event){
        event.preventDefault(); // disable tabulation
        return text + '\u0009';  // add tab

      },

      /*      saveNotes() {
        const parsed = JSON.stringify(this.records);
        localStorage.setItem('records', parsed);
      },*/

      findArrayIndexByDate(date){
        for(let i = 0; i < this.getRecord[this.selectedRecordTypeToShow].length; i++){
          if(date === this.getRecord[this.selectedRecordTypeToShow][i].date)
            return i
        }
        return -1
      },

      editRecord(record){
        this.editedRecord = record;
        //this.editedRecord = this.getRecord[this.selectedRecordTypeToShow][recordId];
        /*let record = this.getRecord[this.selectedRecordTypeToShow][date];
        console.log(record)
        this.editedRecord.recordType  = this.selectedRecordTypeToShow;
        this.editedRecord.date        = record.date;
        this.editedRecord.title       = record.title;
        this.editedRecord.body        = record.body;
        this.editedRecord.enable = true;*/
      },

      saveEditChanges(){
        let editedRecord = this.editedRecord;
        console.log(editedRecord)
        this.$store.dispatch('saveEditRecord', editedRecord);
        this.recordOnEdition = false;
      },
    },

    computed:{
      getRecord(){
        let record = this.$store.getters.RECORDS;
        return record;
      },

      getTypesArray(){
        let array = [];
        for (let i = 0; i < this.recordTypes.length; i++){
          array.push(this.recordTypes[i].name)
        }
        return array
      }
    },

    mounted(){
      this.newNote.date = this.getCurrentDate(); //get date for auto-write to v-date field

      /*if (localStorage.getItem('records')) {
        try {
          this.records = JSON.parse(localStorage.getItem('records'));
        } catch(e) {
          localStorage.removeItem('records');
        }
      }*/
    },

    watch: {
      selectedRecordTypeToShow(){
        this.$store.dispatch('setCurrentRecordType', this.selectedRecordTypeToShow);
        //console.log(this.selectedRecordTypeToShow)
      },

    }
  }
</script>

<style scoped>
  .autoH{
    height: auto;
  }

  .important{
    background-color: red;
  }
  .usial{
    background-color: #4652db;
  }


</style>


