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
        <h3 v-if="this.contributionPercent === 100">
          Unfortunately, you cannot max out your 401k this year
        </h3>
        <div v-else>
          <h3>
            To max out your 401k, you must contribute
            <strong>{{ contributionPercent }}%</strong> of your salary!
          </h3>
          <v-simple-table ref="payTable">
            <template v-slot:default>
              <thead>
                <tr>
                  <th class="text-left">Pay Period</th>
                  <th class="text-left">Amount</th>
                </tr>
              </thead>
              <tbody>
                <tr v-for="item in rows" :key="item.name">
                  <td>{{ item.name }}</td>
                  <td>{{ item.value }}</td>
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
      this.scrollToElement("salary");
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
        value = Number(Math.round(value + "e" + 2) + "e-" + 2);
        rows.push({
          value: this.formatMoney(value),
          name: p.name,
        });
      }

      this.rows = rows;

      if (this.contributionPercent < 100) {
        let field = document.createElement("input");
        field.setAttribute("type", "text");
        document.body.appendChild(field);

        setTimeout(function () {
          field.focus();
          setTimeout(function () {
            field.setAttribute("style", "display:none;");
          }, 50);
        }, 50);
        this.scrollToElement("payTable");
      }
    },
    generateTableRow(dividend, rawContributionPercent) {
      let value = ((rawContributionPercent / 100) * this.salary) / dividend;
      return value;
    },
    scrollToElement(e) {
      const el = this.$refs[e].$el;

      if (el) {
        el.scrollIntoView({ behavior: "smooth" });
      }
    },
  },
};
</script>
