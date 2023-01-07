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

      <v-col cols="8" v-if="this.maxContributionAmount">
        <h3>
          You can contribute {{ formatMoney(maxContributionAmount) }} this year!
        </h3>
        <h3>How much will you make this year?</h3>
        <v-text-field
          ref="salary"
          id="salary"
          v-model="salary"
          label="Salary"
          type="number"
          prefix="$"
          required
          @change="checkBonus"
        ></v-text-field>
        <v-btn color="primary" @click="checkBonus">Next</v-btn>
      </v-col>

      <v-col cols="8" v-if="this.showBonus">
        <h3>Do you want to add money from a bonus?</h3>
        <v-btn color="success" @click="enterBonus" class="mx-2">Yes</v-btn>
        <v-btn color="error" @click="noBonus">No</v-btn>
        <div v-if="this.contributeBonus">
          <v-text-field
            ref="bonus"
            id="bonus"
            v-model="bonus"
            label="Bonus Amount"
            type="number"
            prefix="$"
            required
            @change="determine401kAmount"
          ></v-text-field>
          <v-btn color="primary" @click="determine401kAmount">Submit</v-btn>
        </div>
      </v-col>

      <v-col cols="8" v-if="this.showFinalView && !this.tooMuchMoneyAdded">
        <h3
          v-if="this.contributionPercent === 100"
          ref="badNewsHeader"
          id="badNewsHeader"
        >
          Unfortunately, you cannot max out your 401k this year
        </h3>
        <div v-else>
          <h3>
            To max out your 401k, you must contribute
            <strong
              >{{ contributionPercent }}% ({{ roundedContributionPercent }}%
              <v-tooltip bottom>
                <template v-slot:activator="{ on, attrs }">
                  <v-icon v-bind="attrs" v-on="on">
                    mdi-information-outline
                  </v-icon>
                </template>
                <span
                  >Rounded percent as most employers only allow whole
                  numbers</span
                >
              </v-tooltip>
              )</strong
            >
            of your salary!
          </h3>
          <v-simple-table ref="payTable" id="payTable">
            <template v-slot:default>
              <thead>
                <tr>
                  <th class="text-left">Pay Period</th>
                  <th class="text-left">Amount</th>
                  <th class="text-left">Take Home (Before Taxes)</th>
                </tr>
              </thead>
              <tbody>
                <tr v-for="item in rows" :key="item.name">
                  <td>{{ item.name }}</td>
                  <td>{{ item.value }}</td>
                  <td>{{ item.takeHomeAmount }}</td>
                </tr>
              </tbody>
            </template>
          </v-simple-table>
        </div>
      </v-col>

      <v-col cols="8" v-if="this.tooMuchMoneyAdded">
        <h3 ref="tooMuchMoneyAdded" id="tooMuchMoneyAdded">
          Looks like your bonus covers your entire contribution amount! No need
          to add more through your pay periods!
        </h3>
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
    roundedContributionPercent: 0,
    rows: [],
    showFinalView: false,
    snackbar: false,
    snackbarText: "",
    showBonus: false,
    contributeBonus: false,
    bonus: null,
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

      let amount = 22500;
      if (this.age >= 50) {
        amount += 7500;
      }
      this.maxContributionAmount = amount;
      this.$nextTick(() => this.$refs.salary.focus());
    },
    determine401kAmount() {
      this.snackbar = false;
      this.tooMuchMoneyAdded = false;

      if (!this.salary) {
        this.showSnackbar("Please enter a Salary!");
        return;
      }

      if (Number(this.bonus) > Number(this.salary) || this.bonus < 0) {
        this.showSnackbar("Please enter a valid Bonus Amount!");
        return;
      }

      let adjustedContributionAmount = 0;
      if (this.contributeBonus) {
        adjustedContributionAmount =
          this.maxContributionAmount - Number(this.bonus);
      } else {
        adjustedContributionAmount = this.maxContributionAmount;
      }
      
      const contributionAmount = Number(this.salary);

      const rawContributionPercent =
        (adjustedContributionAmount / contributionAmount) * 100;

      let contributionPercent = Number(
        Math.round(rawContributionPercent + "e" + 3) + "e-" + 3
      );

      let roundedPercent = Math.ceil(rawContributionPercent);
      let roundedContributionPercent = Number(
        Math.round(roundedPercent + "e" + 3) + "e-" + 3
      );

      if (contributionPercent >= 100) {
        this.contributionPercent = 100;
        this.roundedContributionPercent = 100;
      } else if (contributionPercent < 0) {
        this.contributionPercent = 0;
        this.roundedContributionPercent = 0;
        this.tooMuchMoneyAdded = true;
      } else {
        this.contributionPercent = contributionPercent;
        this.roundedContributionPercent = roundedContributionPercent;
      }

      let payPeriods = [
        {
          name: "Weekly",
          value: 52,
        },
        {
          name: "Bi-Weekly",
          value: 26,
        },
        {
          name: "Monthly",
          value: 12,
        },
        {
          name: "Quarterly",
          value: 4,
        },
        {
          name: "Yearly",
          value: 1,
        },
      ];

      let rows = [];
      for (let p of payPeriods) {
        let roundedPercent = Math.ceil(rawContributionPercent);
        let value = this.generateTableRow(p.value, rawContributionPercent);
        let roundedValue = this.generateTableRow(p.value, roundedPercent);

        let takeHomeAmount = this.generateTakeHomeAmount(
          p.value,
          rawContributionPercent
        );

        let roundedTakeHomeAmount = this.generateTakeHomeAmount(
          p.value,
          roundedPercent
        );

        value = Number(Math.round(value + "e" + 2) + "e-" + 2);
        takeHomeAmount = Number(
          Math.round(takeHomeAmount + "e" + 2) + "e-" + 2
        );
        roundedValue = Number(Math.round(roundedValue + "e" + 2) + "e-" + 2);
        roundedTakeHomeAmount = Number(
          Math.round(roundedTakeHomeAmount + "e" + 2) + "e-" + 2
        );
        rows.push({
          takeHomeAmount: `${this.formatMoney(
            takeHomeAmount
          )}  (${this.formatMoney(roundedTakeHomeAmount)})`,
          value: `${this.formatMoney(value)} (${this.formatMoney(
            roundedValue
          )})`,
          name: p.name,
        });
      }

      this.rows = rows;

      this.showFinalView = true;

      if (this.contributionPercent < 100) {
        this.$nextTick(() => this.$refs.salary.blur());
        this.$nextTick(() => this.scrollToElement("payTable"));
      } else {
        this.$nextTick(() => this.scrollToElement("badNewsHeader"));
      }
    },
    generateTableRow(dividend, rawContributionPercent) {
      const value = ((rawContributionPercent / 100) * this.salary) / dividend;
      return value;
    },
    generateTakeHomeAmount(dividend, rawContributionPercent) {
      const takeHomeAmount = this.salary / dividend;
      const deductionAmount =
        ((rawContributionPercent / 100) * this.salary) / dividend;
      let value = takeHomeAmount - deductionAmount;
      return value;
    },
    scrollToElement(e) {
      const el = this.$refs[e].$el.id;
      this.$vuetify.goTo(`#${el}`);
    },
    checkBonus() {
      this.showBonus = true;
    },
    enterBonus() {
      this.contributeBonus = true;
      this.showFinalView = false;
      this.$nextTick(() => this.$refs.bonus.focus());
      this.$nextTick(() => this.scrollToElement("bonus"));
    },
    noBonus() {
      this.contributeBonus = false;
      this.determine401kAmount();
    },
  },
};
</script>
