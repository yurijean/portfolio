<template>
  <div v-show="!hasLoadedFirstMedia" class="spinner"></div>
  <div
    class="project"
    :style="`visibility: ${hasLoadedFirstMedia ? 'visible' : 'hidden'}`"
  >
    <div class="project__heading">
      <h3>
        <slot name="title"></slot>
      </h3>
      <p>
        <slot name="summary"></slot>
      </p>
    </div>

    <div
      class="project__medias"
      @mouseenter="stopSlide()"
      @mouseleave="startSlide()"
    >
      <div class="project__medias__slider">
        <template v-for="media in medias" :key="media.path">
          <div class="project__medias__slider__item">
            <div class="project__medias__slider__item__browser-frame">
              <div></div>
              <div></div>
              <div></div>
            </div>
            <picture>
              <source
                type="image/webp"
                :srcset="
                  require(`../assets/projects/${media.path.replace(
                    /[0-9a-z]+$/i,
                    'webp'
                  )}`)
                "
              />
              <source
                :type="`image/${media.path.match(/[0-9a-z]+$/i)[0]}`"
                :srcset="require(`../assets/projects/${media.path}`)"
              />
              <img
                :src="require(`../assets/projects/${media.path}`)"
                :alt="media.alt"
                loading="lazy"
              />
            </picture>
          </div>
        </template>
      </div>
      <div class="project__medias__slider-controls">
        <div
          :class="
            `project__medias__slider-controls__item ${
              currentMedia === index + 1
                ? 'project__medias__slider-controls__item--active'
                : ''
            }`
          "
          v-for="(media, index) in medias"
          @click="handleSlide(index + 1)"
          :key="media.path"
        ></div>
      </div>
    </div>

    <h4>About this project</h4>
    <p style="margin-bottom: 40px">
      <slot name="about"></slot>
    </p>

    <h4>Technical sheet</h4>
    <ul>
      <slot name="technologies"></slot>
    </ul>
  </div>
</template>

<script>
import { ref, watch, onMounted } from "vue";

export default {
  props: { medias: Array },
  setup(props) {
    let timer;

    const currentMedia = ref(1);
    const currentZIndex = ref(3);
    const slideDirection = ref("right");
    const hasLoadedFirstMedia = ref(false);

    watch(currentMedia, (currentMedia) => {
      const element = document.querySelector(
        `.project__medias__slider__item:nth-child(${currentMedia})`
      );
      currentZIndex.value++;

      element.style.zIndex = currentZIndex.value;
      element.classList.remove("enter-from-right");
      element.classList.remove("enter-from-left");
      void element.offsetWidth;
      element.classList.add(`enter-from-${slideDirection.value}`);
    });

    const handleSlide = (index) => {
      slideDirection.value = index < currentMedia.value ? "left" : "right";
      currentMedia.value = index;
    };

    const startSlide = () => {
      timer = setInterval(() => {
        currentMedia.value =
          currentMedia.value === props.medias.length
            ? 1
            : currentMedia.value + 1;
        slideDirection.value = "right";
      }, 8000);
    };

    const stopSlide = () => {
      clearInterval(timer);
    };

    onMounted(() => {
      const sliderContainer = document.querySelector(
        ".project__medias__slider"
      );
      const sliderContainerChild = document.querySelector(
        ".project__medias__slider__item:nth-child(1)"
      );

      document.querySelector(
        ".project__medias__slider__item:nth-child(1) img, .project__medias__slider__item:nth-child(1) video"
      ).onload = () => {
        sliderContainer.style.height = `${sliderContainerChild.scrollHeight}px`;
        hasLoadedFirstMedia.value = true;
      };

      startSlide();
    });

    return {
      handleSlide,
      stopSlide,
      startSlide,
      currentMedia,
      hasLoadedFirstMedia,
    };
  },
};
</script>

<style>
.spinner {
  margin: 160px auto;
  background-color: var(--background-color);
  height: 90px;
  width: 90px;
  animation: rotate 1s infinite;
}

@keyframes rotate {
  0% {
    transform: rotateZ(0deg);
  }

  100% {
    transform: rotateZ(360deg);
  }
}

.project {
  margin-top: 40px;
  padding: 20px 25px;
  border-radius: 20px;
  background-color: var(--background-color);
  text-align: left;
  overflow: hidden;
  animation: fadeIn 0.4s;
}

@keyframes fadeIn {
  0% {
    transform: translateY(20px);
    opacity: 0.2;
  }

  100% {
    transform: translateY(0px);
    opacity: 1;
  }
}

.project__heading {
  margin-bottom: 40px;
}

.project__medias {
  margin-bottom: 50px;
}

.project__medias__slider {
  position: relative;
  margin-bottom: 30px;
}

.project__medias__slider__item {
  position: absolute;
  left: 0;
}

.project__medias__slider > :first-child {
  z-index: 1;
}

.project__medias__slider > :not(:first-child) {
  z-index: 0;
}

.project__medias__slider .enter-from-left {
  animation: forwards enterFromLeft 0.8s;
}

.project__medias__slider .enter-from-right {
  animation: forwards enterFromRight 0.8s;
}

@keyframes enterFromLeft {
  0% {
    transform: translateX(-100%) scale(1.01);
  }

  85% {
    transform: translateX(0%) scale(1.01);
  }

  100% {
    transform: translateX(0%) scale(1);
  }
}

@keyframes enterFromRight {
  0% {
    transform: translateX(100%) scale(1.01);
  }

  85% {
    transform: translateX(0%) scale(1.01);
  }

  100% {
    transform: translateX(0%) scale(1);
  }
}

.project__medias__slider img,
.project__medias__slider video {
  width: 100%;
}

.project__medias__slider__item__browser-frame {
  background-color: #ddd;
  width: 100%;
  padding: 3px 6px;
  box-sizing: border-box;
  display: flex;
}

.project__medias__slider__item__browser-frame div {
  height: 7px;
  width: 7px;
  border-radius: 50%;
}

.project__medias__slider__item__browser-frame div:not(:first-child) {
  margin-left: 3px;
}

.project__medias__slider__item__browser-frame div:first-child {
  background-color: #e85a54;
}

.project__medias__slider__item__browser-frame div:nth-child(2) {
  background-color: #f7bd2b;
}

.project__medias__slider__item__browser-frame div:last-child {
  background-color: #73cc45;
}

.project__medias__slider-controls {
  display: flex;
  justify-content: center;
}

.project__medias__slider-controls__item {
  width: 12px;
  height: 12px;
  border-radius: 50%;
  background-color: var(--background-color--lighter);
  cursor: pointer;
  transition: transform 0.2s;
}

.project__medias__slider-controls__item:not(:first-child) {
  margin-left: 15px;
}

.project__medias__slider-controls__item:hover:not(.project__medias__slider-controls__item--active) {
  transform: scale(1.25);
}

.project__medias__slider-controls__item--active {
  transform: scale(1.5);
  background-color: var(--background-color--lighter2);
}

@media only screen and (min-width: 992px) {
  .project {
    padding: 40px 50px;
  }

  .project__medias__slider__item__browser-frame {
    padding: 6px 12px;
  }

  .project__medias__slider__item__browser-frame div:not(:first-child) {
    margin-left: 5px;
  }

  .project__medias__slider__item__browser-frame div {
    height: 12px;
    width: 12px;
  }
}
</style>
