<template>
  <div class="event">
    <EventInfomation></EventInfomation>
    <div class="replyInput">
      <ElInput
        v-model="input"
        type="textarea"
        :placeholder="placeholder"
        :autosize="{ minRows: 3 }"
        resize="none"
      >
      </ElInput>
      <button
        class="replyButton"
        :class="{ replyButtonShow: input }"
        @click="reply"
      >
        发送
      </button>
    </div>
    <EventReply
      v-for="message in messages"
      :reply-user-id="message.id"
      :reply-content="message.content"
      :reply-username="message.source.username"
      :reply-user-avatar="message.source.avatar"
      @reply="getReplyUserId(message.source.id)"
    ></EventReply>
  </div>
</template>
<script lang="ts" setup>
const { data } = useAuth();
const route = useRoute();
const { data: messages, refresh } = useFetch(
  `/api/message/event/${route.params.id}`,
);
const input = ref("");
const replyUserId = ref(0);
watch(input, (value) => {
  if (!value) {
    replyUserId.value = 0;
  }
});
const reply = async () => {
  await useFetch(`/api/message/reply`, {
    method: "post",
    query: {
      eventId: route.params.id,
      replyUserId,
      // eslint-disable-next-line @typescript-eslint/no-explicit-any
      sourceId: (data.value?.user as any).id,
    },
    body: { input },
    watch: false,
  });
  input.value = "";
  refresh();
};
const getReplyUserId = (id: number) => {
  replyUserId.value = id;
};
const placeholder = computed(() => {
  if (replyUserId.value) {
    return `回复 ${replyUserId.value} 号用户`;
  }
  return "在此输入你想说的话";
});
</script>
<style scoped>
.event {
  display: flex;
  flex-direction: column;
  align-items: center;
  margin: 20px 0;
}
.event > div {
  box-sizing: border-box;
  width: 65%;
  border: 1px solid var(--el-border-color-light);
  border-radius: 4px;
}
.event > div + div {
  margin-top: 20px;
}
.replyInput {
  position: relative;
  overflow: hidden;
}
.replyButton {
  box-sizing: border-box;
  display: inline-block;
  padding: 6px 12px;
  border: 1px solid var(--el-border-color-light);
  border-bottom-right-radius: 2px;
  transition: all 0.2s ease-in, right 1s ease-in-out;
  position: absolute;
  bottom: 1px;
  overflow: hidden;
  color: black;
  z-index: 1;
  background: transparent;
  cursor: pointer;
}

.replyButton:before {
  content: "";
  position: absolute;
  left: 50%;
  transform: translateX(-50%) scaleY(1) scaleX(1.25);
  top: 100%;
  width: 140%;
  height: 180%;
  background-color: rgba(0, 0, 0, 0.05);
  border-radius: 50%;
  display: block;
  transition: all 0.5s 0.1s cubic-bezier(0.55, 0, 0.1, 1);
  z-index: -1;
}

.replyButton:after {
  content: "";
  position: absolute;
  left: 55%;
  transform: translateX(-50%) scaleY(1) scaleX(1.45);
  top: 180%;
  width: 160%;
  height: 190%;
  background-color: #39bda7;
  border-radius: 50%;
  display: block;
  transition: all 0.5s 0.1s cubic-bezier(0.55, 0, 0.1, 1);
  z-index: -1;
}

.replyButton:hover {
  color: #ffffff;
  border: 1px solid #39bda7;
}

.replyButton:hover:before {
  top: -35%;
  background-color: #39bda7;
  transform: translateX(-50%) scaleY(1.3) scaleX(0.8);
}

.replyButton:hover:after {
  top: -45%;
  background-color: #39bda7;
  transform: translateX(-50%) scaleY(1.3) scaleX(0.8);
}
.replyButtonShow {
  right: 1px;
}
.reply {
  display: flex;
  flex-direction: column;
  height: 120px;
  background: var(--el-fill-color-lighter);
}
.reply > .el-divider--horizontal {
  margin: 0;
}
.replyUser {
  box-sizing: border-box;
  display: flex;
  align-items: center;
  flex: 2;
  padding: 0 10px;
}
.username {
  margin-left: 10px;
  font-weight: bold;
  line-height: 1em;
}
.replyContent {
  box-sizing: border-box;
  flex: 3;
  padding: 10px;
}
</style>
