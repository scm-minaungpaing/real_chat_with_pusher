<template>
    <AppLayout title="Dashboard">
        <template #header>
            <h2 class="font-semibold text-xl text-gray-800 leading-tight">
                <ChatRoomSelection
                    v-if="currentRoom.id"
                    :rooms="chatRooms"
                    :currentRoom="currentRoom"
                    @roomChanged="setRoom( $event )"
                />
            </h2>
        </template>

        <div class="py-12">
            <div class="max-w-7xl mx-auto sm:px-6 lg:px-8">
                <div class="bg-white overflow-hidden shadow-xl sm:rounded-lg">
                    <MessageContainer :messages="messages"/>
                    <InputMessage
                        :room="currentRoom"
                        @messagesent="getMessages()"
                    />
                </div>
            </div>
        </div>
    </AppLayout>
</template>

<script setup>
import AppLayout from '@/Layouts/AppLayout.vue';
import Welcome from '@/Jetstream/Welcome.vue';
import MessageContainer from './messageContainer.vue';
import InputMessage from './inputMessage.vue';
import { onMounted, watch } from 'vue'
import { reactive, ref } from '@vue/reactivity';
import axios from 'axios';
import ChatRoomSelection from './chatRoomSelection.vue';


const chatRooms = ref([])
const currentRoom = ref([])
const messages = ref([])

const getRooms = async () => {
    await axios.get('/chat/rooms').then( res => {
        chatRooms.value = res.data;
        setRoom(res.data[0])
    })
    .catch( error => {
        console.log(error)
    })
}

const setRoom = ( room ) => {
    currentRoom.value = room
    // getMessages()
}

const getMessages = () => {
    axios.get(`./chat/room/${currentRoom.value.id}/messages`)
    .then( res => {
        messages.value = res.data
    })
    .catch(error => {
        console.log(error)
    })
}

const connect = () => {
    if ( currentRoom.value) {
        getMessages()
        window.Echo.channel(`chat.${currentRoom.value.id}`)
        .listen('.my-chat-event', e => {
            getMessages();
        })
    }
}

const disconnect = ( room ) => {
    window.Echo.leave(`chat.${room.id}`)
}

watch(currentRoom, (val, oldVal) => {
    if ( oldVal.id) {
        disconnect( oldVal )
    }
    connect()
})

onMounted(async () => {
    await getRooms()
})
</script>
