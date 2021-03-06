<template>
  <vue-wrapping-lightbox />
</template>

<script setup lang="ts">
import { defineComponent, h, useSlots, ref, useAttrs } from "vue";
import NextArrow from "./components/NextArrow.vue";
import PreviousArrow from "./components/PreviousArrow.vue";
import Close from "./components/Close.vue";

const slots = useSlots();

const isActive = ref(false);
const activeIndex = ref(0);

const previousImage = (event) => {
  activeIndex.value === 0
    ? (activeIndex.value = slots.default().at(0).children.length - 1)
    : (activeIndex.value -= 1);
  event.stopPropagation();
};

const nextImage = (event) => {
  activeIndex.value === slots.default().at(0).children.length - 1
    ? (activeIndex.value = 0)
    : (activeIndex.value += 1);
  event.stopPropagation();
};

const closeLightbox = () => (isActive.value = false);

const openImage = (index: number): void => {
  isActive.value = true;
  activeIndex.value = index;
};

const attrs = useAttrs();

const VueWrappingLightbox = defineComponent({
  render() {
    if (!slots.default) {
      return undefined;
    }
    return [
      h(
        "div",
        {
          ...attrs,
        },
        slots.default()
      ),
      [
        h(
          "div",
          {
            class: { lightbox: true, "lightbox-active": isActive.value },
            onClick: closeLightbox,
          },
          [
            h("div", { class: "lightbox-content" }, [
              h(slots.default().at(0).children.at(activeIndex.value), {
                class: "lightbox-image",
              }),
            ]),
            h(
              "div",
              { class: "lightbox-close", onClick: closeLightbox },
              h(Close)
            ),
            h(
              "div",
              { class: "lightbox-previous", onClick: previousImage },
              h(PreviousArrow)
            ),
            h(
              "div",
              { class: "lightbox-next", onClick: nextImage },
              h(NextArrow)
            ),
            h(
              "div",
              { class: "lightbox-counter" },
              `${activeIndex.value + 1} / ${
                slots.default().at(0).children.length
              }`
            ),
            h(
              "div",
              { class: "lightbox-thumbnails" },
              slots.default().map((vnode, index) => {
                if (vnode.type === "img") {
                  return h("div", {
                    class: {
                      "lightbox-thumbnail": true,
                      "lightbox-thumbnail-active": activeIndex.value === index,
                    },
                    style: {
                      "background-image": `url(${vnode.props.src})`,
                    },
                    onClick: (event) => {
                      openImage(index);
                      event.stopPropagation();
                    },
                  });
                }
                return vnode.children
                  ? vnode.children.map((vnode, index) =>
                      h("div", {
                        class: {
                          "lightbox-thumbnail": true,
                          "lightbox-thumbnail-active":
                            activeIndex.value === index,
                        },
                        style: {
                          "background-image": `url(${vnode.props.src})`,
                        },
                        onClick: (event) => {
                          openImage(index);
                          event.stopPropagation();
                        },
                      })
                    )
                  : vnode;
              })
            ),
          ]
        ),
      ],
    ];
  },
});

defineExpose({
  openImage,
});
</script>

<style>
.lightbox {
  display: none;
  position: fixed;
  top: 0;
  bottom: 0;
  left: 0;
  right: 0;
  height: 100%;
  width: 100%;
  background: rgba(0, 0, 0, 0.85);
}

.lightbox-active {
  display: flex;
  align-items: center;
  justify-content: center;
}

.lightbox-previous,
.lightbox-next,
.lightbox-close {
  position: absolute;
  background: rgba(0, 0, 0, 0.2);
  width: 25px;
  height: 25px;
  cursor: pointer;
}

.lightbox-previous {
  top: 50%;
  left: 0;
  padding: 10px 10px 10px 0px;
  border-top-right-radius: 5px;
  border-bottom-right-radius: 5px;
}

.lightbox-next {
  top: 50%;
  right: 0;
  padding: 10px 0 10px 10px;
  border-top-left-radius: 5px;
  border-bottom-left-radius: 5px;
}

.lightbox-close {
  top: 0;
  right: 0;
  padding: 10px;
  border-bottom-left-radius: 5px;
}

.lightbox-counter {
  position: absolute;
  background: rgba(0, 0, 0, 0.2);
  bottom: 0;
  right: 0;
  color: #ffffff;
  font-family: sans-serif;
  padding: 10px;
  border-top-left-radius: 5px;
}

.lightbox-image {
  width: 100%;
  height: 100%;
  max-width: 100%;
  max-height: 100vh;
}

.lightbox-thumbnails {
  background: rgba(0, 0, 0, 0.2);
  text-align: center;
  white-space: nowrap;
  position: fixed;
  display: flex;
  border-bottom-left-radius: 5px;
  border-bottom-right-radius: 5px;
  top: 0;
  padding: 10px;
  justify-content: space-between;
  gap: 10px;
}

.lightbox-thumbnail {
  display: inline-block;
  width: 50px;
  height: 50px;
  cursor: pointer;
  background-size: cover;
  background-position: center center;
}
</style>
