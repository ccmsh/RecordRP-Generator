<template>
    <div class="navbar bg-base-100 shadow-sm">
        <a class="btn btn-ghost text-xl">レコードを置き換えるためのサイト</a>
    </div>
    <div style="margin: 2%;">
        <p class="text-lg font-semibold ml-2">置き換えるレコードを選択</p>
        <div>
            <label v-for="record in records" :key="record" class="btn btn-square p-1 ml-2"
                :class="{ 'btn-active btn-info': selected.includes(record) }" style="cursor:pointer;">
                <input type="checkbox" class="hidden" :id="record" :value="record" v-model="selected">
                <img :src="`/records/${record}.png`" :alt="record">
            </label>
        </div>
        <div class="ml-2 mt-2 collapse bg-base-100 border-base-300 border">
            <input type="checkbox" />
            <div class="collapse-title font-semibold">選んだディスク</div>
            <div class="collapse-content text-sm">
                <p class="ml-2" v-for="select in selected">
                    {{ select }}を選択しました
                </p>
            </div>
        </div>
    </div>
</template>
<script>
export default {
    data() {
        return {
            records: [
                '5',
                '11',
                '13',
                'blocks',
                'cat',
                'chirp',
                'far',
                'mall',
                'mellohi',
                'stal',
                'strad',
                'wait',
                'ward',
                'pigstep',
                'otherside',
                'relic',
                'creator_diskbox',
            ],
            selected: []
        }
    }
}
</script>