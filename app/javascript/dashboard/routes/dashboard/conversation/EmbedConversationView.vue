<script setup>
import { computed, onMounted, watch } from 'vue';
import { useStore } from 'dashboard/composables/store';
import { BUS_EVENTS } from 'shared/constants/busEvents';
import { emitter } from 'shared/helpers/mitt';
import ConversationBox from 'dashboard/components/widgets/conversation/ConversationBox.vue';

const props = defineProps({
  conversationId: {
    type: [String, Number],
    default: 0,
  },
  showContactPanel: {
    type: Boolean,
    default: false,
  },
});

const store = useStore();

const currentChat = computed(() => store.getters.getSelectedChat);
const chatList = computed(() => store.getters.getAllConversations);

const findConversation = () => {
  const id = parseInt(props.conversationId, 10);
  return chatList.value.find(chat => chat.id === id);
};

const setActiveChat = () => {
  const selectedConversation = findConversation();
  if (
    !selectedConversation ||
    selectedConversation.id === currentChat.value.id
  ) {
    return;
  }
  store.dispatch('setActiveChat', { data: selectedConversation }).then(() => {
    emitter.emit(BUS_EVENTS.SCROLL_TO_MESSAGE, {});
  });
};

const loadConversation = () => {
  if (!props.conversationId) {
    return;
  }
  if (findConversation()) {
    setActiveChat();
  } else {
    store.dispatch('getConversation', props.conversationId);
  }
};

watch(() => props.conversationId, loadConversation);
watch(() => chatList.value.length, setActiveChat);

onMounted(() => {
  store.dispatch('setActiveInbox', 0);
  store.dispatch('agents/get');
  store.dispatch('inboxes/get');
  store.dispatch('teams/get');
  loadConversation();
});
</script>

<template>
  <section class="flex w-full h-full min-w-0 bg-n-background">
    <ConversationBox
      is-on-expanded-layout
      :is-contact-panel-open="showContactPanel"
    />
  </section>
</template>
