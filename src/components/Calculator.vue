<template>
  <v-container>
    <v-row class="text-center" align="center" justify="center">
      <v-col cols="8">
        <h2>How old are you?</h2>
        <v-text-field
          v-model="age"
          label="Age"
          type="number"
          required
          @change="determineMaxContributionAmount"
        ></v-text-field>
        <v-btn color="primary" @click="determineMaxContributionAmount"
          >Next</v-btn
        >
      </v-col>

      <v-col cols="8" v-show="this.maxContributionAmount">
        <h3>
          You can contribute {{ formatMoney(maxContributionAmount) }} this year!
        </h3>
        <h3>How much will you make this year?</h3>
        <v-text-field
          ref="salary"
          v-model="salary"
          label="Salary"
          type="number"
          prefix="$"
          required
          @change="determine401kAmount"
        ></v-text-field>
        <v-btn color="primary" @click="determine401kAmount">Submit</v-btn>
      </v-col>
      
      <v-col cols="8" v-show="this.contributionPercent"> 
        <h3 v-if="this.contributionPercent === 100">Unfortunately, you cannot max out your 401k this year</h3>
        <h3 v-else>To max out your 401k, you must contribute <strong>{{contributionPercent}}%</strong> of your salary!</h3>

        <!-- TODO table for how much that is per paycheck -->
      </v-col>

      <v-snackbar v-model="snackbar">
        {{ snackbarText }}

        <template v-slot:action="{ attrs }">
          <v-btn color="primary" text v-bind="attrs" @click="snackbar = false">
            Close
          </v-btn>
        </template>
      </v-snackbar>
    </v-row>
  </v-container>
</template>

<script>
export default {
  name: "Calculator",
  data: () => ({
    age: null,
    maxContributionAmount: 0,
    salary: null,
    contributionPercent: 0,
    snackbar: false,
    snackbarText: "",
  }),
  methods: {
    showSnackbar(message) {
      this.snackbarText = message;
      this.snackbar = true;
    },
    formatMoney(number) {
      number = number.toString();
      number = number.replace(/\B(?=(\d{3})+(?!\d))/g, ",");
      return `$${number}`;
    },
    determineMaxContributionAmount() {
      this.snackbar = false;

      if (!this.age) {
        this.showSnackbar("Please enter an Age!");
        return;
      }

      if (this.age < 18) {
        this.showSnackbar("You must be over 18 to have a 401k");
        return;
      }

      let amount = 19500;
      if (this.age >= 50) {
        amount += 6500;
      }
      this.maxContributionAmount = amount;
      this.$nextTick(() => this.$refs.salary.focus());
    },
    determine401kAmount() {
      this.snackbar = false;

      if (!this.salary) {
        this.showSnackbar("Please enter a Salary!");
        return;
      }

      const contributionAmount = Number(this.salary);

      const rawContributionPercent =
        (this.maxContributionAmount / contributionAmount) * 100;


      let contributionPercent = Number(
        Math.round(rawContributionPercent + "e" + 3) + "e-" + 3
      );

      if(contributionPercent >= 100){
        this.contributionPercent = 100
      }else{
        this.contributionPercent = contributionPercent;
      }

    },
  },
};
</script>
