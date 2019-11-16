<template>
  <div>
    <h1 class="text-md-center">Данные ВИЭ “Вертушка”</h1>

    <v-container grid-list-md class="px-3">
      <v-layout row wrap>
        <v-flex md12>
          <v-card>
            <v-card-text>
              <v-container grid-list-md class="px-3">
                <v-layout row wrap>
                  <v-flex md12>
                    <b>Маяк</b>:
                    <a
                      :href="`https://etherscan.io/address/${lighthouse.address}`"
                      target="blank"
                    >{{ lighthouse.name }}</a>
                  </v-flex>
                </v-layout>
              </v-container>
              <v-divider/>

              <v-container grid-list-md class="px-3">
                <v-layout row wrap>
                  <v-flex md12>
                    <div>
                      <b>Адрес владельца ВИЭ в сети</b>: <a :href="`https://etherscan.io/address/${account}`" target="blank" >{{ account }}</a>
                    </div>
                    <div>
                      <b>Стоимость</b>: {{ price | fromWei(9, 'XRT') }} |
                      <b>Баланс</b>:
                      <a :href="`https://etherscan.io/token/${token}?a=${account}`" target="blank" >{{ balance | fromWei(9, 'XRT') }}</a>
                      <span v-if="price > 0">| Approved: {{ approveTrade | fromWei(9, 'XRT') }}</span>
                    </div>
                  </v-flex>
                </v-layout>
              </v-container>

              <v-btn
                v-if="price > 0 && approveTrade < price"
                :disabled="loadingApprove || balance < price"
                @click="approve">
                  Approve
              </v-btn>

              <v-btn v-if="approveTrade >= price" @click="demand">
                Заявка
              </v-btn>

              <v-btn v-if="approveTrade >= price" @click="offer">
                Ответ
              </v-btn>
            </v-card-text>
          </v-card>
        </v-flex>

        <v-flex md12>
          <v-card v-if="liability.length>0">
            <v-card-title primary-title>
              <div>
                <h3 class="headline mb-0">Показания с 01 по 30 мая 2019 года</h3>
              </div>
            </v-card-title>
            <v-card-text v-for="(item, i) in liability" :key="i">
                <b>Суммарная генерация: 100 МВт</b>
                <br>
                <b>Лог от счётчика:</b>
                <a :href="`https://ipfs.io/ipfs/${resultLink}`" target="_blank">{{ resultLink }}</a>
                <br>

                <b>Отправить показания на проверку</b>
                <br>
                <v-btn @click="result(item.address)" :disabled="resultSent ? true : false" >Отправить</v-btn>
                <br>

                <br>
                <b>модель:</b>
                {{ item.model }}
                <br>
                <b>параметры:</b>
                {{ item.objective }}
                <br>
                <b>токен:</b>
                {{ item.token }}
                <br>
                <b>стоимость:</b>
                {{ item.cost }}
                <br>
                <b>заказчик:</b>
                {{ item.promisee }}
                <br>
                <b>исполнитель:</b>
                {{ item.promisor }}
                <br>
                <div v-if="item.result != ''">
                  <b>Результат:</b>
                  <a :href="`https://ipfs.io/ipfs/${item.result}`" target="_blank">{{ item.result }}</a>
                  <span v-if="item.check === true">+</span>
                  <span v-else>-</span>
                </div>
                <div v-if="item.result == ''">
                  <b>Результат:</b>
                  <v-btn @click="result(item.address)">Отправить</v-btn>
                </div>
              </v-card-text>
          </v-card>
        </v-flex>
      </v-layout>

      <v-layout row wrap>
        <v-flex md6>
          <v-card>
            <v-card-title primary-title>
              <div>
                <h3 class="headline mb-0">Заявки</h3>
              </div>
            </v-card-title>
            <v-card-text>
              <p class="t-break" v-for="(item, i) in demands" :key="i">
                <b>отправитель:</b>
                {{ item.sender }}
                <br>
                <b>валидатор:</b>
                {{ item.validator }}
                <br>
                <b>модель:</b>
                {{ item.model }}
                <br>
                <b>параметры:</b>
                {{ item.objective }}
                <br>
                <b>токен:</b>
                {{ item.token }}
                <br>
                <b>стоимость:</b>
                {{ item.cost }}
                <br>
                <b>дедлайн:</b>
                {{ item.deadline }}
              </p>
            </v-card-text>
          </v-card>
        </v-flex>

        <v-flex md6>
          <v-card>
            <v-card-title primary-title>
              <div>
                <h3 class="headline mb-0">Предложения</h3>
              </div>
            </v-card-title>
            <v-card-text>
              <p class="t-break" v-for="(item, i) in offers" :key="i">
                <b>отправитель:</b>
                {{ item.sender }}
                <br>
                <b>валидатор:</b>
                {{ item.validator }}
                <br>
                <b>модель:</b>
                {{ item.model }}
                <br>
                <b>параметры:</b>
                {{ item.objective }}
                <br>
                <b>токен:</b>
                {{ item.token }}
                <br>
                <b>стоимость:</b>
                {{ item.cost }}
                <br>
                <b>дедлайн:</b>
                {{ item.deadline }}
              </p>
            </v-card-text>
          </v-card>
        </v-flex>
      </v-layout>
    </v-container>
</div>
</template>

<script>
import Vue from "vue";
import * as config from "../config";

export default {
  data() {
    return {
      lighthouse: {
        name: "",
        address: ""
      },
      account: "",
      price: config.PRICE,
      token: "",
      balance: 0,
      approveTrade: 0,
      loadingApprove: false,
      liability: [],
      demands: [],
      offers: [],
      resultSent: false,
      resultLink: config.RESULT,
      nonce: 0
    };
  },
  mounted() {
    this.lighthouse.name = this.$robonomics.lighthouse.name;
    this.lighthouse.address = this.$robonomics.lighthouse.address;
    this.account = this.$robonomics.account.address;
    this.token = this.$robonomics.xrt.address;
    this.$robonomics.factory.call
      .nonceOf(this.$robonomics.account.address)
      .then(r => {
        this.nonce = Number(r);
      });
    this.$robonomics.onDemand(config.MODEL, msg => {
      const item = this.demands.find(item => item.signature === msg.signature);
      if (!item) {
        this.demands = [{ ...msg }, ...this.demands.slice(0, 10)];
      }
    });
    this.$robonomics.onOffer(config.MODEL, msg => {
      const item = this.offers.find(item => item.signature === msg.signature);
      if (!item) {
        this.offers = [{ ...msg }, ...this.offers.slice(0, 10)];
      }
    });
    this.$robonomics.onResult(msg => {
      this.setResult(msg.liability, msg.result, false);
    });
    this.$robonomics.onLiability((err, liability) => {
      const item = this.liability.find(
        item => item.address === liability.address
      );
      if (!item) {
        liability.getInfo().then(info => {
          this.liability = [
            {
              address: liability.address,
              worker: liability.worker,
              ...info
            },
            ...this.liability
          ];
        });
        liability.onResult().then(result => {
          this.setResult(liability.address, result, true);
        });
      }
    });
    this.fetchBalance();

    // FOR TEST PURPOSE ONLY
    this.liability.push({
      "address": "0x911Ea2bE315f4dEDbc8C457eB9A1234971f59A81",
      "lighthouse": "0x202a09A451DE674d2d65Bf1C90968a8d8F72cf7b",
      "validator": "0x17B82177D8753bd8090dadA60B953CFaDD9eF492",
      "model": "QmUB6ajZTLLMZg7re1v4hw44aoG8HDQDHr9JyujU264Aw2",
      "objective": "Qmbm3o2wkqseSEi5F69CPAuDrsKnrwTJ3HN5FVLPgLHKUm",
      "token": "0x966ebbfd7ecbcf44b1e05341976e0652cfa01360",
      "cost": 0,
      "promisee": "0x4af74a76aA7B934C7397FDD0C2428762c8F7c550",
      "promisor": "0xEb51Cf2a474BfC756cD40A9e9092E6eEe15f2dc3",
      "result": ""
    });
  },
  methods: {
    fetchBalance() {
      return this.$robonomics.xrt.call
        .balanceOf(this.$robonomics.account.address)
        .then(balanceOf => {
          this.balance = balanceOf;
          if (balanceOf > 0) {
            return this.$robonomics.xrt.call.allowance(
              this.$robonomics.account.address,
              this.$robonomics.factory.address
            );
          }
          return false;
        })
        .then(allowance => {
          if (allowance) {
            this.approveTrade = allowance;
          }
        });
    },
    approve() {
      this.loadingApprove = true;
      return this.$robonomics.xrt.send
        .approve(this.$robonomics.factory.address, this.price, {
          from: this.$robonomics.account.address
        })
        .then(() => {
          this.loadingApprove = false;
          return this.fetchBalance();
        })
        .catch(() => {
          this.loadingApprove = false;
        });
    },
    setResult(address, result, check = true) {
      const i = this.liability.findIndex(item => item.address === address);
      if (i >= 0) {
        Vue.set(this.liability, i, { ...this.liability[i], result, check });
      }
    },
    demand() {
      this.$robonomics.web3.eth.getBlock("latest", (e, r) => {
        const demand = {
          model: config.MODEL,
          objective: config.OBJECTIVE,
          token: this.$robonomics.xrt.address,
          cost: this.price,
          lighthouse: this.$robonomics.lighthouse.address,
          validator: config.VALIDATOR,
          validatorFee: 0,
          deadline: r.number + 1000,
          nonce: this.nonce
        };
        this.nonce++;
        this.$robonomics.sendDemand(demand);
      });
    },
    offer() {
      this.$robonomics.web3.eth.getBlock("latest", (e, r) => {
        const offer = {
          model: config.MODEL,
          objective: config.OBJECTIVE,
          token: this.$robonomics.xrt.address,
          cost: this.price,
          lighthouse: this.$robonomics.lighthouse.address,
          validator: config.VALIDATOR,
          lighthouseFee: 0,
          deadline: r.number + 1000,
          nonce: this.nonce
        };
        this.nonce++;
        this.$robonomics.sendOffer(offer);
      });
    },
    result(address) {
      this.resultSent = true;
      this.$robonomics.sendResult({
        liability: address,
        success: true,
        result: config.RESULT
      });
    }
  }
};
</script>

<style>
.block {
  border: 1px solid #eee;
  padding: 10px;
  margin: 10px 0;
}
</style>
