<template>
  <canvas ref="canvas"></canvas>
</template>

<script setup>
import { onBeforeUnmount, onMounted, ref, watch } from "vue-demi";
import * as PIXI from "pixi.js";
import { Spine } from "pixi-spine";
import { useLanguage } from "@/store/languageStore";
import { storeToRefs } from "pinia";

let app = null;
const canvas = ref(null);
const animationSkin = ref("");
const animationInst = ref(null);

const createPixi = () => {
  app = new PIXI.Application({
    view: canvas.value,
    transparent: true,
    antialias: true,
    resolution: 1,
    width: 95,
    height: 95,
  });
  app.loader
    .add("spineCharacter", "./chest/box.json")
    .load((loader, resources) => {
      animationInst.value = new Spine(resources.spineCharacter.spineData);
      app.stage.addChild(animationInst.value);
      animationInst.value.state.setAnimation(0, animationSkin.value, true);
      const localRect = animationInst.value.getLocalBounds();

      animationInst.value.scale.set(0.3, 0.3);
      animationInst.value.position.set(-localRect.x / 3, -localRect.y / 3);
      canvas.value.style.touchAction = "auto";
      app.start();
    });
};

const clearPixiCache = () => {
  Object.keys(PIXI.utils.BaseTextureCache).forEach((item) => {
    PIXI.utils.BaseTextureCache[item].textureCacheIds = [];
    PIXI.utils.BaseTextureCache[item].textureCacheIds = null;
    delete PIXI.utils.BaseTextureCache[item];
  });

  Object.keys(PIXI.utils.TextureCache).forEach((item) => {
    PIXI.utils.TextureCache[item].textureCacheIds = [];
    PIXI.utils.TextureCache[item].textureCacheIds = null;
    delete PIXI.utils.TextureCache[item];
  });
  // PIXI.utils.clearTextureCache()

  app.destroy(true);
  app = null;
};

const languageStore = useLanguage();
const { curLang } = storeToRefs(languageStore);

watch(curLang, () => {
  // if (curLang.value.name === "zhtw" || curLang.value.name === "ch") {
  //   animationInst.value.state.setAnimation(0, "login", true);
  // } else {
  //   animationInst.value.state.setAnimation(0, "login_EN_VN_PH", true);
  // }
});

onMounted(() => {
  // if (curLang.value.name === "zhtw" || curLang.value.name === "ch") {
  //   animationSkin.value = "login";
  // } else {
  //   animationSkin.value = "login_EN_VN_PH";
  // }
  animationSkin.value = "box";
  createPixi();
});

onBeforeUnmount(() => {
  clearPixiCache();
  animationInst.value = null;
  // app.destroy(true)
  // app = null
});
</script>

<style></style>
