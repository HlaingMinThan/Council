<template>
    <div id="replies">
        <h2 class="text-2xl text-green-500 ml-2 my-5">Replies</h2>
        <New-Reply @store="store" v-if="!isLocked"></New-Reply>
        <p v-else class="text-center text-red-600 mt-5 font-semibold">
            This Thread Has Been Locked By Admin! No More Replies Allowed Yet...
        </p>

        <div class="bg-grey-300  border-gray-300">
            <div v-if="allReplies.length">
                <div v-for="reply in allReplies" :key="reply.id">
                    <Reply :reply="reply" @destroy="destroy"></Reply>
                </div>
            </div>
            <div v-else>
                <h1 class="ml-2">No Replies Yet !!!</h1>
            </div>
        </div>
        <Pagination
            :paginationDatas="paginationDatas"
            @pageChanged="pageChanged"
        ></Pagination>
    </div>
</template>
<script>
import Reply from "./Reply.vue";
import NewReply from "./NewReply.vue";
import axios from "axios";
export default {
    props: ["lock"],
    components: { Reply, NewReply },
    data() {
        return {
            allReplies: [],
            paginationDatas: [],
            isLocked: this.lock
        };
    },
    methods: {
        destroy(replyId) {
            this.$emit("destroy");
            this.allReplies = this.allReplies.filter(reply => {
                return reply.id != replyId;
            });
        },
        store(newReply) {
            // this.allReplies.unshift(newReply);
            this.fetchDatas(1);
            document
                .querySelector("#replies")
                .scrollIntoView({ behavior: "smooth" });
            this.$emit("store");
        },
        // every time a user click pagination link
        pageChanged(pageNum) {
            this.fetchDatas(pageNum);
            document
                .querySelector("#replies")
                .scrollIntoView({ behavior: "smooth" });
        },
        // this method run initial first time
        fetchDatas(pageNum) {
            // pageNum is empty on initial fetch
            //pageNum get From pageChanged event from pagination component
            if (!pageNum) {
                // check user search From url with page query
                let query = location.search.match(/page=(\d+)/);
                pageNum = query ? query[1] : 1;
            }
            axios
                .get(`${location.pathname}/replies?page=${pageNum}`)
                .then(res => {
                    this.allReplies = res.data.data;
                    this.paginationDatas = res.data;
                });
        }
    },
    created() {
        this.fetchDatas(); //initial fetching replies and pagination datas

        window.events.$on("lockThread", () => {
            this.isLocked = true;
        });
        window.events.$on("unLockThread", () => {
            this.isLocked = false;
        });
    }
};
</script>

<style></style>
