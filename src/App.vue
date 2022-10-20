<script setup>
  import { ref, computed, reactive } from 'vue';

  const formatter = new Intl.NumberFormat("en-US", {
    style: 'currency', 
    currency: 'USD',
    minimumFractionDigits: 0,
    maximumFractionDigits: 0,
  });      
  const dateProgramStarted = new Date(2022, 3, 1, 0, 0);
  let pricePerDay = ref(0);
  let unpaidDays = ref(0);
  let dateDiscountStarted = ref();
  let attends = reactive([
    {d:1, attends: true}, 
    {d:2, attends: true}, 
    {d:3, attends: true}, 
    {d:4, attends: true}, 
    {d:5, attends: true},
  ]);
  const total = computed(() => {
    let discountedPrice = 0.75 * pricePerDay.value;
    let dailyRefund  = discountedPrice < 12 ? pricePerDay.value - 12 : 0.25 * pricePerDay.value;
    
    let total = Math.max(0, dailyRefund) * totalDays.value;     
    return formatter.format(total);
  });
  
  const totalDays = computed(() => {    
    let timestamp = Date.parse(dateDiscountStarted.value);
    if (isNaN(timestamp)) {
      let now = new Date();
      timestamp = now.getTime();
    }
    let discountStarted = new Date(timestamp);
    return getWeekdaysCount(dateProgramStarted, discountStarted);
  });

  function updateAttends(index) {
    attends[index].attends = !attends[index].attends;
  }
  function getWeekdaysCount(startDate, endDate) {
    let count = 0;
    const curDate = new Date(startDate.getTime());
    while (curDate <= endDate) {
        const dayOfWeek = curDate.getDay();
        let daysAttending = attends.filter((element, id) => {return element.attends;})
        let daysIndexAttending = daysAttending.map((element, id) => {return element.d;})
        if(dayOfWeek !== 0 && dayOfWeek !== 6 && daysIndexAttending.includes(dayOfWeek) ) count++;
        curDate.setDate(curDate.getDate() + 1);
    }
    return count - unpaidDays.value;
  }
</script>

<template>
  <div>
    <h1>Estimate how much you will be refunded from CWELCC?</h1>
    <p class='step'>
      Enter your current daycare price per day.    
      <input type="number" v-model="pricePerDay">
    </p>    
    <p class='step'>
    Enter the day your fees were reduced by 25%.
    <input type="date" name="" id="" v-model="dateDiscountStarted">
    </p>
    <p class='step'>
      Enter the amount of unpaid days between April 1st and the date your fees were reduced.
      <input type="number" v-model="unpaidDays">
    </p>
    <div class='step'>
      <p>Select which days your child attends daycare</p>
      <div class="buttons">
        <button :class="{'attends': attends[0].attends}" @click="updateAttends(0)">Monday</button>
        <button :class="{'attends': attends[1].attends}" @click="updateAttends(1)">Tuesday</button>
        <button :class="{'attends': attends[2].attends}" @click="updateAttends(2)">Wednesday</button>
        <button :class="{'attends': attends[3].attends}" @click="updateAttends(3)">Thursday</button>
        <button :class="{'attends': attends[4].attends}" @click="updateAttends(4)">Friday</button>      
      </div>
    </div>    
  </div>  
  <div class='step total'>Amount to receive: <span class="red">{{total}}</span></div>
  <p class="note">*Not official, or binding; possibly inaccurate; for presentation purposes only.</p>
  <p>© 2022 | Richard Pitul <a href="https://patreon.com/richpitul" target="_blank" rel="noopener noreferrer"><img width="20" src="./assets/patreon.png" alt=""></a></p>
</template>

<style  lang="scss" scoped>
  h1 {
    color:#C2E7DA;
    line-height: 1;
  }
  input {
    text-align: center;
  }
  button.attends {
    background:#BAFF29;
    color: #1A1B41;
  }
  .buttons {
    display:flex;
    gap:8px;
    justify-content: center;
    flex-direction: column;
    @media (min-width:500px) {
      flex-direction: row;
    }
    button {
      min-width: 125px;
      text-transform: uppercase;
      border:none;      
    }
  }
  .total {
    font-size: 1.25rem;
    text-transform: uppercase;
    span {
      color:#C2E7DA;
    }
  }
  .step {
    padding:15px 0;
    &:nth-child(2n) {
      background:#1A1B41;
    }
  }
  a {
    vertical-align: middle;
  }
</style>
