<template>
  <div class="btnGroup">
    <div :class="[mode == 'today' ? 'L1' : 'L2']" @click="changeDate('today')">
      {{ $t("gameRecord.today") }}
    </div>
    <div
      :class="[mode == 'yesterday' ? 'M1' : 'M2']"
      @click="changeDate('yesterday')"
    >
      {{ $t("gameRecord.yesterday") }}
    </div>
    <div
      :class="[mode == 'thisweek' ? 'M1' : 'M2']"
      @click="changeDate('thisweek')"
    >
      {{ $t("gameRecord.七日") }}
    </div>
    <div
      :class="[mode == 'lastweek' ? 'R1' : 'R2']"
      @click="changeDate('lastweek')"
    >
      {{ $t("gameRecord.十四日") }}
    </div>
  </div>
  <!-- <table>
    <thead class="playerInfo">
      <tr>
        <td></td>
        <td></td>
        <td></td>
        <td v-price="totalbet" class="totalbet"></td>
        <td v-price="totalValidbet" class="totalValidbet"></td>
        <td
          v-price="totalWinLose"
          class="totalWinLose"
          :style="{ color: totalWinLose / 10000 < 0 ? '#c80000' : 'green' }"
        ></td>
      </tr>
    </thead>
  </table> -->
  <div class="content">
    <table>
      <thead class="listTitle">
        <tr class="playerData">
          <td></td>
          <td></td>

          <td v-price="totalbet" class="totalbet"></td>
          <td v-price="totalValidbet" class="totalValidbet"></td>
          <td
            v-price="totalWinLose"
            class="totalWinLose"
            :style="{ color: totalWinLose / 10000 < 0 ? '#c80000' : 'green' }"
          ></td>
          <td></td>
        </tr>

        <tr>
          <td>{{ $t("gameRecord.time") }}</td>
          <td>{{ $t("gameRecord.game") }}</td>
          <td>{{ $t("gameRecord.totalBets") }}</td>
          <td>{{ $t("gameRecord.有效投注") }}</td>
          <td>{{ $t("gameRecord.winLose") }}</td>
          <td></td>
        </tr>
      </thead>
      <tbody>
        <template v-for="(v, k) in tradeRecord_Data" :key="k + 'k'">
          <tr :class="[{ odd: k % 2 === 0 }, { even: k % 2 !== 0 }]">
            <td>{{ new Date(v.created_at).toLocaleString("en-GB") }}</td>
            <td>
              {{ codeToName(v.game_code, v.studio) }}
            </td>
            <td
              v-price="v.bet"
              :style="{ color: v.bet < 0 ? '#c80000' : '' }"
            ></td>
            <td v-price="v.valid_bet"></td>
            <td
              v-price="v.winlose"
              :style="{ color: v.winlose < 0 ? '#c80000' : 'green' }"
            ></td>
            <td>
              <span
                v-if="
                  v.studio == 'goldenwind' ||
                  v.studio == 'joygames' ||
                  v.studio == 'jili'
                "
                @click="godist2(v)"
                >></span
              >
            </td>
          </tr>
        </template>
      </tbody>
    </table>
  </div>
  <!-- <div class="rankContentbottom">
    <div class="first">
      <div class="rankWord">
        <span>{{ $t("summary") }}:</span>
      </div>
      <span></span>
    </div>
    <div class="second">{{}}</div>
  </div> -->
  <iframe :src="iframeurl" v-if="iframeurl"></iframe>
  <img
    src="@/assets/images/modal/closurel_button.png"
    class="closeIframe"
    @click="iframeurl = ''"
    v-if="iframeurl && !distIframe_studio.includes('goldenwind')"
  />
  <!--  -->
</template>

<script setup>
import { ref, onMounted, computed } from "vue-demi";
// import { useStore } from "@/store";
// import { storeToRefs } from "pinia";
// const { useAuth } = useStore();
// const { userUuid } = storeToRefs(useAuth());
import { getPlayerId } from "@/utils/cookie";
import request from "@/utils/request.js";
import {
  getotherplatformGameList,
  electrongamesList,
} from "./otherplatformGame.js";
const distIframe_studio = ref();
const tradeRecord_Data = ref();
const mode = ref("today");
const iframeurl = ref("");

// 初始 是今天
const startTime = ref(new Date(new Date().setHours(0, 0, 0, 0)).toISOString());
const endTime = ref(new Date().toISOString());
const totalbet = computed(() => {
  let totalbetData = 0;
  if (tradeRecord_Data.value != null) {
    tradeRecord_Data.value.forEach((el) => {
      totalbetData += el.bet;
    });
  }
  return totalbetData;
});
const totalValidbet = computed(() => {
  let totalbetData = 0;
  if (tradeRecord_Data.value != null) {
    tradeRecord_Data.value.forEach((el) => {
      totalbetData += el.valid_bet;
    });
  }
  return totalbetData;
});
const totalWinLose = computed(() => {
  var totalData = 0;
  if (tradeRecord_Data.value != null) {
    tradeRecord_Data.value.forEach((el) => {
      totalData += el.winlose;
    });
  }
  return totalData;
});
const changeDate = (e) => {
  mode.value = e;
  var startdate = new Date();
  var enddate = new Date();
  var date = new Date();
  switch (e) {
    case "today":
      startdate.setHours(0, 0, 0, 0);
      break;
    case "yesterday":
      startdate.setDate(date.getDate() - 1);
      startdate.setHours(0, 0, 0, 0);
      enddate.setDate(date.getDate() - 1);
      enddate.setHours(23, 59, 59, 59);
      break;
    case "thisweek":
      startdate.setDate(date.getDate() - 7);
      startdate.setHours(0, 0, 0, 0);
      // enddate.setDate(date.getDate());
      // enddate.setHours(23, 59, 59, 59);
      break;
    case "lastweek":
      startdate.setDate(date.getDate() - 14);
      startdate.setHours(0, 0, 0, 0);
      // enddate.setDate(date.getDate());
      // enddate.setHours(23, 59, 59, 59);
      break;
  }
  startTime.value = new Date(startdate).toISOString();
  endTime.value = new Date(enddate).toISOString();
  getdata();
};
//
const electronGamesData = {}; //電子遊戲 key對應的注單及遊戲名稱+
var otherPlatformGameList = {};
// 哈希遊戲gameCode轉gameName
const gameNameToHASHWord = (e) => {
  var nextHtml;
  switch (e) {
    case 15:
      nextHtml = "Super Niu Niu";
      break;
    case 16:
      nextHtml = "Hash Baccarat";
      break;
    case 17:
      nextHtml = "Odd or Even";
      break;
    case 18:
      nextHtml = "Dragon & Tiger";
      break;
    case 19:
      nextHtml = "Hash Football";
      break;
    case 20:
      nextHtml = "Hash Roulette";
      break;
    case 21:
      nextHtml = "Hash Xoc Dia";
      break;
    case 22:
      nextHtml = "Hash Tien Gow";
      break;
    case 23:
      nextHtml = "Banker Niu Niu";
      break;
    case 24:
      nextHtml = "World Cup PSO";
      break;
    case 27:
      nextHtml = "Roulette 16";
      // 1214
      break;
    case 26:
      nextHtml = "Hash Horse Racing";
      break;
    case 28:
      nextHtml = "Cockfight";
      break;
    case 25:
      nextHtml = "Fish, Prawn, Crab";
      break;
  }
  return nextHtml;
};

// 打開細單
const godist2 = async (e) => {
  // 鏈遊
  distIframe_studio.value = e.studio;
  let urls = `/games/${e.studio}/detail_record/${e.round_uuid}?lang=en-US`;
  const { data } = await request()({
    url: urls,
    method: "get",
  });
  iframeurl.value = data.data.url;
  // if (e.studio == "goldenwind") {
  //   let url = process.env.VUE_APP_DIST;
  //   iframeurl.value =
  //     url +
  //     `?gameName=${gameNameToHASHWord(
  //       Number(e.game_code)
  //     )}&gamePlayerId=${getPlayerId()}&language=en_US&id=${
  //       e.round_uuid
  //     }&mode=backend`;
  // } else {
  //   // zh-TW
  //   // en-US
  //   let urls = `/games/${e.studio}/detail_record/${e.round_uuid}?lang=en-US`;
  //   const { data } = await request()({
  //     url: urls,
  //     method: "get",
  //   });
  //   iframeurl.value = data.data.url;
  // }

  // if (e.studio == "goldenwind") {
  //   let url = process.env.VUE_APP_DIST;
  //   iframeurl.value =
  //     url +
  //     `?gameName=${gameNameToHASHWord(
  //       Number(e.game_code)
  //     )}&gamePlayerId=${getPlayerId()}&language=en_US&id=${
  //       e.round_uuid
  //     }&mode=backend`;
  // } else if (e.studio == "joygames") {
  //   // http://192.168.88.217/orderid/?orderid=11xCaptainMoneyx49338x37&gamecode=Fish_CaptainMoney&language=en
  //   // var s = vm.$store.state.language.replace("_", "").toLowerCase();
  //   var s = "zh_TW";
  //   if (s == "enph") {
  //     s = "en";
  //   } else if (s == "zhtw") {
  //     s = "zhcn";
  //   } else if (s == "enus") {
  //     s = "en";
  //   }
  //   s = "en";
  //   var dist2 = process.env.VUE_APP_ELECTRONGAMEDIST;
  //   iframeurl.value = `${dist2}?orderid=${e.round_uuid}&gamecode=${
  //     electronGamesData[e.game_code].dist_gamecode
  //   }&language=${s}`;
  // }
};
// 搜尋資料
const getdata = async () => {
  const { data } = await request()({
    url: `/member/game_log?user_uuid=${getPlayerId()}&start_time=${
      startTime.value
    }&end_time=${endTime.value}`,
    method: "get",
  });
  tradeRecord_Data.value = data.data.data;
};

const codeToName = (e, gametype) => {
  if (otherPlatformGameList[gametype] == undefined) {
    return "PP-Game";
  } else if (otherPlatformGameList[gametype][e] == undefined) {
    return "PP-Game";
  } else {
    if (gametype == "goldenwind") {
      return gameNameToHASHWord(Number(e)); //哈希遊戲
    } else if (gametype == "joygames") {
      return electronGamesData[e].name.en; // electrongame
    } else {
      return otherPlatformGameList[gametype][e].gameName.en;
    }
  }
};
// 把array轉成json
// 0307
const electronArrayDataTOJSON = () => {
  for (let i = 0; i < electrongamesList().length; i++) {
    const el = electrongamesList()[i];
    electronGamesData[el.gameid] = el;
  }
};
electronArrayDataTOJSON();
// 得到吉利 遊戲清單
const get_other_Platform_GameList = () => {
  var dada = getotherplatformGameList();
  dada.forEach((el) => {
    // if (!otherPlatformGameList[el.studio][el.studio_game_id]) {
    //   otherPlatformGameList[el.studio][el.studio_game_id].gameName = {};
    //   otherPlatformGameList[el.studio][el.studio_game_id].gameName.en = "";
    //   otherPlatformGameList[el.studio][el.studio_game_id].gameName.tw = "";
    //   otherPlatformGameList[el.studio][el.studio_game_id].gameName.cn = "";
    // }
    // if (
    //   el.studio !== "goldenwind" &&
    //   el.studio !== "joygames" &&
    //   el.studio !== "funky" &&
    //   el.studio !== "evo" &&
    //   el.studio !== "pg" &&
    //   el.studio !== "facai" &&
    //   el.studio !== "jili" &&
    //   el.studio !== "PP" &&
    //   el.studio !== "mgplus"
    // ) {
    //   otherPlatformGameList[el.studio] = {};
    //   otherPlatformGameList[el.studio][el.studio_game_id] = {};
    //   otherPlatformGameList[el.studio][el.studio_game_id].gameName = {};
    //   otherPlatformGameList[el.studio][el.studio_game_id].gameName.en = "";
    //   otherPlatformGameList[el.studio][el.studio_game_id].gameName.tw = "";
    //   otherPlatformGameList[el.studio][el.studio_game_id].gameName.cn = "";
    // }

    // if (el.studio !== "goldenwind" && el.studio !== "joygames") {
    if (otherPlatformGameList[el.studio] == undefined) {
      otherPlatformGameList[el.studio] = {};
      otherPlatformGameList[el.studio][el.studio_game_id] = {};
      otherPlatformGameList[el.studio][el.studio_game_id].gameName = {};
      otherPlatformGameList[el.studio][el.studio_game_id].gameName.en = "";
      otherPlatformGameList[el.studio][el.studio_game_id].gameName.tw = "";
      otherPlatformGameList[el.studio][el.studio_game_id].gameName.cn = "";
    }

    otherPlatformGameList[el.studio][el.studio_game_id] = {};
    otherPlatformGameList[el.studio][el.studio_game_id].gameName = {};
    // otherPlatformGameList[el.studio_game_id].studio = el.studio;
    otherPlatformGameList[el.studio][el.studio_game_id].gameName.en =
      el.game_name_en;
    otherPlatformGameList[el.studio][el.studio_game_id].gameName.tw =
      el.game_name_tw;
    otherPlatformGameList[el.studio][el.studio_game_id].gameName.cn =
      el.game_name_cn;
    // }
  });
};
get_other_Platform_GameList();
window.addEventListener("message", (e) => {
  if (e.data === "closeWebview") iframeurl.value = "";
});
onMounted(() => {
  getdata();
});
</script>

<style lang="scss" scoped>
.btnGroup {
  width: 50%;
  box-sizing: border-box;
  display: flex;
  justify-content: center;
  align-items: center;
  margin: 3% 3% 0;
  div {
    padding: 0.5rem 1rem;
    width: 25%;
    background-size: 100% 100%;
    background-repeat: no-repeat;
    color: white;
    display: inline-block;
    font-size: 1rem;
    word-break: keep-all;
    text-align: center;
  }
  .L2 {
    background: $my-func-button-bg;
    border: $my-func-button-border 1px solid;
    border-radius: 5px 0 0 5px;
  }
  .L1 {
    background: $anotherservice-btn-bg;
    border: transparent 1px solid;
    border-radius: 5px 0 0 5px;
  }
  .M2 {
    background: $my-func-button-bg;
    border: $my-func-button-border 1px solid;
  }
  .M1 {
    background: $anotherservice-btn-bg;
    border: transparent 1px solid;
  }
  .R2 {
    background: $my-func-button-bg;
    border: $my-func-button-border 1px solid;
    border-radius: 0px 5px 5px 0px;
  }
  .R1 {
    background: $anotherservice-btn-bg;
    border: transparent 1px solid;
    border-radius: 0px 5px 5px 0px;
  }
}
.content {
  max-height: 800px;
  overflow-y: scroll;
  &::-webkit-scrollbar {
    display: block;
    // height: 10px;
    width: 10px;
  }
  &::-webkit-scrollbar-track {
    background: $scrollbar-track-bg;
  }
  &::-webkit-scrollbar-thumb {
    background: $scrollbar-thumb-bg;
    border-radius: 20px;
  }

  table {
    padding: 2% 2% 0;
    margin: auto;
    width: calc(100%);
    border-collapse: separate;
    border-spacing: 0;
    text-align: center;
    font-size: 0.8rem;
    // color: #2d4c76;
    word-break: keep-all;
    // border-collapse: collapse;
    position: relative;

    thead {
      td {
        padding-top: 0.5rem;
        padding-bottom: 0.5rem;
      }
    }

    .playerInfo {
      tr {
        display: flex;
        justify-content: end;
        // .totalbet {
        //   width: 23%;
        // }

        // .totalValidbet {
        //   width: 16%;
        // }
        // .totalWinLose {
        //   width: 14%;
        // }
      }
    }
    .listTitle {
      background: $my-func-button-bg;
      color: rgb(180, 150, 227);
      .playerData {
        background: $leaderBoard-table-even-bg;
      }
    }
    tr {
      width: 100%;
      height: 3rem;
      line-height: 3rem;
      .rotate90 {
        transform: rotate(90deg);
      }
      // td {
      //   padding: 0.25rem 0;
      // }
      // td:nth-child(1) {
      //   width: 25%;
      // }
      // td:nth-child(2) {
      //   width: 25%;
      // }
      // td:nth-child(3) {
      //   width: 25%;
      // }
      // td:nth-child(4) {
      //   width: 15%;
      // }
      // td:nth-child(5) {
      //   width: 7%;
      //   padding-right: 0.5rem;
      // }
    }
    .odd {
      background: $func-button-bg-selected;
      height: 50px;
      line-height: 50px;
    }
    .even {
      background: $my-func-button-bg;
      height: 50px;
      line-height: 50px;
    }

    tfoot {
      position: sticky;
      width: 100%;
      font-size: 1rem;
      bottom: 4rem;
      height: 4rem;
      color: white;
      tr {
        width: 100%;
        height: 100%;
      }
    }
  }
}

iframe {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  z-index: 999;
  background: $popout-page-bg;
}
.closeIframe {
  position: fixed;
  // img
  left: 0;
  top: 0;
  width: 45px;
  object-fit: contain;
  z-index: 1000;
}
</style>
