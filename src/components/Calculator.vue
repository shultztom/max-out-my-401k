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
          @change="determine401kAmount"
        ></v-text-field>
        <v-btn color="primary" @click="determine401kAmount">Submit</v-btn>
      </v-col>

      <v-col cols="8" v-if="this.showFinalView">
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
            <strong>{{ contributionPercent }}%</strong> of your salary!
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
    rows: [],
    showFinalView: false,
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

      if (contributionPercent >= 100) {
        this.contributionPercent = 100;
      } else {
        this.contributionPercent = contributionPercent;
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
        let value = this.generateTableRow(p.value, rawContributionPercent);
        let takeHomeAmount = this.generateTakeHomeAmount(
          p.value,
          rawContributionPercent
        );
        value = Number(Math.round(value + "e" + 2) + "e-" + 2);
        takeHomeAmount = Number(
          Math.round(takeHomeAmount + "e" + 2) + "e-" + 2
        );
        rows.push({
          takeHomeAmount: this.formatMoney(takeHomeAmount),
          value: this.formatMoney(value),
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
  },
};
</script>
