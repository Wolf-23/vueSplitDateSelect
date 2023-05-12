<template>
  <!-- Creo tre select con rispettivi option (di cui uno hidden in modo che non sia selezionabile, usato per specificare lo scopo di ogni singola select), nelle quali recupero il valore del giorno, mese ed anno selezionati, utilizzando il v-model
  Quando il valore selezionato cambia, viene chiamata la funzione changeDate.
  -->
  <div class="container">
    <div>
      <h2>Seleziona una data:</h2>
      <select v-model="selectedDay" @change="changeDate">
        <option value="" hidden>Giorno</option>
        <option v-for="day in days" :key="day" :value="day">{{ day }}</option>
      </select>
      <select v-model="selectedMonth" @change="changeDate">
        <option value="" hidden>Mese</option>
        <option v-for="(month, index) in months" :key="month" :value="index + 1">{{ month }}</option>
      </select>
      <select v-model="selectedYear" @change="changeDate">
        <option value="" hidden>Anno</option>
        <!-- I valori dell'array years sono calcolati nella generateYears -->
        <option v-for="year in years" :key="year" :value="year">{{ year }}</option>
      </select>
      <!-- Se dateText vale 'e' (Valore assegnato dalla changeDate, riga 79) viene stampata una stringa di errore, altrimenti la data selezionata -->
      <h2 v-if="dateText == 'e'"> La data inserita non è valida</h2>
      <h2 v-else-if="dateText"> La data selezionata è: {{ dateText }}</h2>
    </div>
  </div>
</template>

<script>
// Importo MomentJs
import moment from 'moment';

export default {
  name: 'DateSelect',
  data() {
    return {
      selectedDay: '',
      selectedMonth: '',
      selectedYear: '',
      days: [],
      months: moment.months(),
      years: [],
      dateText: ''
    };
  },
  // Quando viene montato il componente, chiamo i metodi generateYears e updateDays 
  mounted() {
    this.generateYears()
    this.updateDays();
  },
  methods: {
    // Calcolo l'anno corrente e l'anno di partenza poi tramite un ciclo for pusho tutti i valori compresi tra entrambi dentro l'array years.
    generateYears() {
      const currentYear = moment().year();
      const minYear = 1900;
      for (let i = currentYear; i >= minYear; i--) {
        this.years.push(i);
      }
    },
    updateDays() {
      // Recupero mese ed anno selezionati
      this.days = [];
      const month = this.selectedMonth - 1;
      const year = this.selectedYear;
      const monthIsValid = month || month == 0;
      // Se i valori di mese ed anno non sono validi, emetto una data nulla
      if(!monthIsValid || !year) {
        this.$emit('date-selected', null);
        return
      }
      // Tramite la daysInMonth(funzione di Moment) calcolo quanti giorni ci sono nel mese selezionato dell'anno selezionato (per anni bisestili), e pusho tutti i valori dentro l'array days
      let daysInMonth = moment({month, year}).daysInMonth();
      for (let i = 1; i <= daysInMonth; i++) {
        this.days.push(i);
      }
      // Controllo che il giorno sia selezionato, e che in caso sia maggiore del risultato di daysInMonth, svuoto il valore del giorno selezionato ed emetto una data nulla
      if (this.selectedDay && this.selectedDay > daysInMonth) {
        this.selectedDay = '';
        this.$emit('date-selected', null);
      }
    },
    changeDate() {
      // Se tutti e tre i valori sono selezionati, tramite moment mi salvo la data dentro una const
      if (this.selectedDay && this.selectedMonth && this.selectedYear) {
        const date = moment(`${this.selectedDay}-${this.selectedMonth}-${this.selectedYear}`, 'DD-MM-YYYY');
        // Se è una data valida emetto la data con il valore selezionato (ho deciso di formattarla per poterla stampare in modo più leggibile)
        if (date.isValid()) {
          this.$emit('date-selected', date.toDate());
          this.dateText = date.format('D MMM YYYY');
        } else {
          // Altrimenti svuoto il valore del giorno selezionato, emetto una data vuota ed assegno il valore 'e' a dateText, in modo da poterla usare per stampare una messaggio di errore dentro il template
          this.selectedDay = '';
          this.$emit('date-selected', null);
          this.dateText = 'e';
        }
        // Se non sono selezionati tutti e tre i valori, emetto una data vuota
      } else {
        this.$emit('date-selected', null);
      }
      this.updateDays();
    }
  }
};
</script>

<style scoped>

.container {
  height: 100vh;
  width: 100vw;
  display: flex;
  justify-content: center;
  align-items: center;
  background-color: aliceblue;
}

select {
  margin: 15px 7px;
  height: 50px;
  width: 200px;
  padding: 5px;
  border-radius: 5px;
}

h2 {
  text-align: center;
  margin: 5px;
}

</style>
