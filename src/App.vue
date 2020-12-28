<template>
  <div id="main-app" class="container">
    <div class="row justify-content-center">
      <add-appointment @add="addItem"/>
      <searchapt @searchRecords="searchAppointments" />
      <appointments-list :appointments="filterApts" @remove="removeItem" @edit="editItem"/>
    </div>
  </div>
</template>

<script>
import Searchapt from './components/Searchapt.vue';
import AddAppointment from './components/AddAppointment';
import AppointmentsList from './components/AppointmentsList';
import axios from "axios";
import _ from "lodash";



export default {
  name: 'MainApp',
  data: function(){
    return{
      title: "Appointment List",
      appointments: [],
      aptIndex: 0,
      searchTerms: "",
      filterKey: "petName",
      filterDir: "asc"
    };
  },
  components: {
    AppointmentsList,
    AddAppointment,
    Searchapt
  },
  mounted() {
    axios
      .get("./data/appointments.json")
      .then(response => (this.appointments = response.data.map(item => {
        item.aptId = this.aptIndex;
        this.aptIndex++;  
        return item
      })));
  },
  computed: {
    searchedApts: function() {
      return this.appointments.filter(item => {
        return(
          item.petName.toLowerCase().match(this.searchTerms.toLowerCase()) ||
          item.petOwner.toLowerCase().match(this.searchTerms.toLowerCase()) ||
          item.aptNotes.toLowerCase().match(this.searchTerms.toLowerCase()) 
        );
      });
    },
    filterApts: function() {
        return _.orderBy(
            this.searchedApts,
            item => {
              return item[this.filterKey].toLowerCase();
            }, this.filterDir
        )
    }
  },
  methods: {
    searchAppointments: function(terms) {
      this.searchTerms = terms;
    },
    addItem: function(apt) {
      apt.aptId = this.aptIndex;
      this.aptIndex++;
      this.appointments.push(apt);
    },
    removeItem: function(apt){
      this.appointments = _.without(this.appointments, apt);
    },

    editItem: function(id, field, text) {
      const aptIndex = _.findIndex(this.appointments, {
        aptId: id
      })
      this.appointments[aptIndex] [field] = text;
    }
  },
};
</script>

