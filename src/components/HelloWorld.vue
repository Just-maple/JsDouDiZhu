<template>
  <div style="text-align: center">
    <div>
      倍数:{{cards_list.base}}
    </div>

    <!--地主牌展示区-->
    <div style="display: inline-block">
      <card :key="card" v-for="card in cards_list.bonusCards" :card="card"></card>
      <div style="clear: both"></div>
    </div>

    <!--出牌展示区-->
    <div style="margin: 40px">
      <div v-if="cards_list.player_now.handler">player{{cards_list.player_now.handler}}
        <br>
        {{typeCaller(getCardsType(cards_list.cards_now))}}
      </div>
      <div style="display: inline-block;margin-top: 20px">
        <card v-for="card,i in cards_list.cards_now"
              :key="card"
              :style="`${i>0?'margin-left:-25px':''}`"
              :card="card">
        </card>
        <div style="clear: both"></div>
      </div>
    </div>


    <!--玩家牌展示区-->
    <div v-for="player in 3" style="margin-top: 20px">
      <div style="margin: 20px" :style="`${cards_list.dizhu.on===player?'color:red':''}`">
        {{cards_list.player_now.player===player?'⭐️':''}}
        player{{player}} <span v-if="cards_list.dizhu.on">{{cards_list.dizhu.on===player?'地主':'农民'}}</span>
        {{getCards(player).length}}张
      </div>
      <div style="display: inline-block">
        <card @click.native="selectCard(player,card)"
              v-for="card,i in getCards(player)"
              :key="card"
              :player_select="getSelectCards(player)"
              :style="`${i>0?'margin-left:-25px':''}`"
              :card="card">
        </card>
        <div style="clear: both"></div>
        <div v-if="cards_list.player_now.player===player" style="margin-top: 20px">
          <div v-if="!cards_list.dizhu.on">
            <div>
              <button @click="competeDizhu(player,1)"> {{cards_list.dizhu.compete_pre?'抢':'叫'}}地主</button>
              <button @click="competeDizhu(player,0)">不{{cards_list.dizhu.compete_pre?'抢':'叫'}}</button>
            </div>
          </div>
          <div v-else>

            {{typeCaller(getCardsType(cards_list[`player${player}_selected`]))}}
            <button v-if="checkCards(player)" @click="pushCards(player)">出牌</button>
            <button v-if="cards_list.cards_now.length && cards_list.player_now.handler!==player"
                    @click="nextTick">不要
            </button>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
  import Card from "./card";

  export default {
    name: 'HelloWorld',
    components: {Card},
    data() {
      return {
        cards: [],
        dlist_tmp: 'AAKKQQJJ101099887766554433',
        list_tmp: 'AKQJ109876543',
        tlist_tmp: 'AAAKKKQQQJJJ101010999888777666555444333',
        cards_num: ['3', '4', '5', '6', '7', '8', '9', '10', 'J', 'Q', 'K', 'A', '2', 'Joker'],
        cards_list: {
          base: 1,
          player1: [],
          player1_selected: [],
          player2: [],
          player2_selected: [],
          player3: [],
          player3_selected: [],
          bonusCards: [],
          cards_now: [],
          player_now: {
            handler: '',
            player: ""
          },
          dizhu: {
            competitors: [],
            compete_start: "",
            compete_now: "",
            compete_pre: "",
            on: ""
          },
        },
        back: ""
      }
    },
    created() {
      this.back = JSON.stringify(this.cards_list);
      this.initGame()
    },
    methods: {
      initGame() {
        this.cards = [];
        this.restore();
        this.initCards();
        this.shuffleCards();
        this.splitCards();
        this.startGame();
      },
      restore() {
        this.cards_list = JSON.parse(this.back)
      },
      competeDizhu(player, compete) {
        let dizhu = this.cards_list.dizhu;
        let cs = dizhu.competitors;
        let pi = cs.indexOf(player);
        let on_dizhu = (player) => {
          dizhu.on = player;
          let plist = this.getCards(player)
          plist.push(...this.cards_list.bonusCards)
          plist.sort(this.card_compare)
          this.cards_list.player_now.player = player;
          return
        };
        if (!compete) {
          cs.splice(pi, 1);
          pi = pi === cs.length ? 0 : pi;
          this.cards_list.player_now.player = cs[pi];
          //没人叫地主
          if (!cs.length) {
            this.initGame();
            return
          }
          if (dizhu.compete_now === cs[pi]) {
            on_dizhu(cs[pi]);
            return
          }
          if (dizhu.compete_pre) {
            dizhu.compete_pre = cs[0]
          }
          return
        }

        if (cs.length === 1) {
          on_dizhu(player);
          return
        }
        if (dizhu.compete_pre) {
          this.cards_list.base *= 2;
          if (player === dizhu.compete_pre) {
            on_dizhu(player);
            return
          }
        } else {
          dizhu.compete_pre = player
        }
        dizhu.compete_now = player;
        pi = pi === cs.length - 1 ? 0 : pi + 1;
        this.cards_list.player_now.player = cs[pi]

      },
      nextTick() {
        let player = this.cards_list.player_now.player
        this.cards_list[`player${player}_selected`] = []
        this.cards_list.player_now.player = this.getNextPlayer(player)
      },
      getCards(player) {
        return this.cards_list[`player${player}`]
      },
      getSelectCards(player) {
        return this.cards_list[`player${player}_selected`]
      },
      checkCards(player) {
        let cards = this.getSelectCards(player).slice(0).sort(this.card_compare)
        let cards_now = this.cards_list.cards_now
        let type = this.getCardsType(cards)
        let type_now = this.getCardsType(cards_now)
        if (!type) {
          return false
        }
        if (!this.cards_list.player_now.handler || this.cards_list.player_now.handler === player) {
          return type
        }
        if (type_now[0] === 'rocket') {
          return false
        }
        if (type[0] === 'rocket') {
          return type
        }
        if (type[0] === 'xxxx' && type_now[0] !== 'xxxx') {
          return type
        }
        if (type[0] === type_now[0] && (this.card_num_compare(type[1], type_now[1]) < 0)) {
          return type
        }
      },
      isList(cards) {
        return this.list_tmp.indexOf(cards.join('')) > -1 && cards.length >= 5
      },
      isDoubleList(cards) {
        let i = this.dlist_tmp.indexOf(cards.join(''))
        return i >= 0 && i % 2 === 0 && cards.length >= 6
      },
      isTriList(cards) {
        let i = this.tlist_tmp.indexOf(cards.join(''))
        return i >= 0 && i % 3 === 0 && cards.length >= 6
      },
      isSame(cards) {
        let flag = true
        cards.map(card => {
          if (!flag) {
            return
          }
          if (card !== cards[0]) {
            flag = false
          }
        })
        return flag
      },
      typeCaller(type) {
        if (!type) {
          return
        }
        let s = type[0]
        let n = type[1]
        switch (true) {
          case s.indexOf('dlist') >= 0: {
            return '连对'
          }
          case s.indexOf('list') >= 0: {
            return '顺子'
          }
          case s.indexOf('xxxyyy') >= 0: {
            return '飞机'
          }
          default:
            return {
              'xx': '对' + n,
              'rocket': '火箭',
              'xxx': '三个' + n,
              'xxxa': '三带一',
              'xxxaa': '三带二',
              'xxxx': '炸弹',
              'xxxxab': '四带二',
              'xxxxaabb': '四带两对',
            }[s] || n
        }
      }, isOtherPairs(s, i0, i1) {
        for (let i = 0; i < s.length - 1; i++) {
          if (i === i0) {
            i = i1 - 1
            continue
          }
          if (s[i] !== s[i + 1]) {
            return false
          }
          i += 1
        }
        return true
      },
      getCardsType(cards) {
        let s = []
        cards = cards.slice(0)
        cards.sort(this.card_compare)
        cards.map(card => {
          s.push(card.slice(0, card.length - 1))
        });
        let checkCombo = (s, step, combo_method, pair = false) => {
          let isComboAndPairs = (s, combo_method, i0, i1) => {
            return combo_method(s.slice(i0, i1)) && this.isOtherPairs(s, i0, i1)
          };
          let xyzw = {3: 'xxx', 4: 'xxxx', 6: 'xxxyyy', 9: 'xxxyyyzzz', 12: 'xxxyyyzzzwww', 16: 'xxxyyyzzzwwwttt'};
          let ab = pair ? 'aabbccdd' : 'abcde';
          for (let i = 0; i < s.length - step + 1; i++) {
            if (pair ? isComboAndPairs(s, combo_method, i, i + step) : combo_method(s.slice(i, i + step))) {
              return [xyzw[step] + ab.slice(0, s.length - step), s[i]]
            }
          }
          return false
        };
        let checkComboAndPairs = (s, step, combo_method) => {
          return checkCombo(s, step, combo_method, true)
        };
        let checkSameType = (s) => {
          if (this.isSame(s)) {
            if (s[0] === 'Joker') {
              return ['rocket', s[0]]
            } else {
              return [new Array(s.length + 1).join('x'), s[0]]
            }
          }
        };
        let type = false;
        switch (s.length) {
          case 1:
            type = ['x', s[0]];
            break;
          case 4: {
            type = checkSameType(s);
            if (!type) {
              type = checkCombo(s, 3, this.isSame)
            }
            break
          }
          case 6: {
            type = checkCombo(s, 6, this.isTriList)
            if (!type) {
              type = checkCombo(s, 4, this.isSame)
            }
            break
          }
          case 8: {
            type = checkComboAndPairs(s, 4, this.isSame)
          }
          case 9: {
            if (this.isTriList(s)) {
              type = ['xxxyyyzzz', s[0]]
            }
            break
          }
          default: {
            if (type) {
              break
            }
            if (s.length < 4) {
              type = checkSameType(s)
            }
            [4, 5].map(i => {
              if (s.length % i === 0) {
                type = (i === 4 ? checkCombo : checkComboAndPairs)(s, s.length * 3 / i, this.isTriList)
              }
            })
          }
        }
        if (type) {
          return type
        } else if (this.isList(s)) {
          return ['list' + s.length, s[0]]
        } else if (this.isDoubleList(s)) {
          return ['dlist' + s.length, s[0]]
        } else {
          return false
        }

      },
      pushCards(player) {
        let cards = this.getSelectCards(player)
        let type = this.checkCards(player)
        if (!type) {
          alert("你的牌不合规矩或不够大")
          return
        }
        if (type[0] === 'xxxx' || type[0] === 'rocket') {
          this.cards_list.base *= 2
        }
        let all_cards = this.getCards(player)
        cards.map(card => {
          all_cards.splice(all_cards.indexOf(card), 1)
        })
        if (!all_cards.length) {
          alert(player + "赢了")
          this.initGame()
          return
        }
        this.cards_list[`player${player}_selected`] = []
        this.cards_list.cards_now = cards.slice(0).sort(this.card_compare)

        this.cards_list.player_now.handler = player
        this.nextTick()
      },
      getNextPlayer(player) {
        player += 1;
        if (player > 3) {
          player = 1
        }
        return player
      },
      startGame() {
        this.cards_list.player_now.player = 1
        // this.cards_list.player_now.player = Math.floor(Math.random() * 3) + 1
        let dizhu = this.cards_list.dizhu
        dizhu.compete_start = this.cards_list.player_now.player;
        [0, 1, 2].map(i => {
          let t = this.cards_list.player_now.player + i
          dizhu.competitors.push(t > 3 ? t - 3 : t)
        })

      },
      selectCard(player, card) {
        let p_cl_sele = this.cards_list[`player${player}_selected`]
        if (p_cl_sele.indexOf(card) >= 0) {
          p_cl_sele.splice(p_cl_sele.indexOf(card), 1)
        } else {
          p_cl_sele.push(card)
        }
      },
      shuffleCards() {
        let randomsort = (a, b) => {
          return Math.random() > .5 ? -1 : 1;
        };
        this.cards.sort(randomsort);
      },
      card_num_compare(a, b) {

        return this.cards_num.indexOf(b) - this.cards_num.indexOf(a)
      },
      card_compare(a, b) {
        let a0 = a.slice(0, a.length - 1);
        let b0 = b.slice(0, b.length - 1);
        let s = this.cards_num.indexOf(b0) - this.cards_num.indexOf(a0)
        if (s) {
          return s
        } else {
          a0 = a[a.length - 1]
          b0 = b[b.length - 1]
          return a0 - b0
        }
      },
      splitCards() {
        this.cards_list.bonusCards = this.cards.slice(0, 3).sort(this.card_compare)
        this.cards_list.player1 = this.cards.slice(3, 20).sort(this.card_compare)
        this.cards_list.player2 = this.cards.slice(20, 37).sort(this.card_compare)
        this.cards_list.player3 = this.cards.slice(37).sort(this.card_compare)
      },
      initCards() {
        this.cards_num.map(num => {
          for (let color in [0, 1, 2, 3])
            if (num === 'Joker' && color > 1) {
              break
            } else {
              this.cards.push(num + color)
            }
        })
      }
    }
  }
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
  button {
    padding: 5px 20px;
    border: 1px solid;
    border-radius: 5px;
    background: #fff;
    margin-bottom: 15px;
  }

  button:hover {
    color: dodgerblue;
  }

  button:active {
    color: dodgerblue;
  }
</style>
