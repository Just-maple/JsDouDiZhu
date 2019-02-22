<template>
  <div class="card"
       :style="`color:${card[card.length-1]%2===1?'red':''};height:${height}px;
       ${(player_select||[]).indexOf(card)>=0?'margin-top:-10px':''}`"
  >
    <div>{{getCardName(card)}}</div>
    <div style="float: right"
         :style="`margin-top:${height/2}px;`"
    >
      {{getCardName(card,true)}}
    </div>
    <div style="clear: both"></div>
  </div>
</template>

<script>
  export default {
    name: "card",
    data() {
      return {
        height: 80
      }
    },
    created() {
    },
    props: ['card', 'player_select'],
    methods: {
      getCardName(card, bottom = false) {
        let num = card.slice(0, card.length - 1)
        let flower_index = card[card.length - 1]
        let flower = num === 'Joker' ? '' : '♠♥♣♦'[flower_index]
        return bottom ? flower + num : num + flower
      },
    }
  }
</script>

<style scoped>
  .card {
    padding: 0 2px;
    width: 50px;
    border: 1px solid;
    border-radius: 2px;
    background: #fff;
    text-align: left;
    float: left;
  }

</style>
