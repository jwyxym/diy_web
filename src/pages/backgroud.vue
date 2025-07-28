<template>
    <view v-for = 'i in pics' :id = 'i.code' class = 'pic'>{{ i.base64 }}</view>
</template>
<script setup lang = 'ts'>
    import { ref, onBeforeMount, reactive } from 'vue';
    import SQL from '../script/sql.ts'
    import { YuGiOhCard, Card } from '../script/Yu-Gi-Oh-Cards.ts';
    import { exportImage } from '../script/pic.js'
    let pics = reactive([] as Array<{
        base64 : string,
        code : string
    }>);
    const load = async () => {
            pics.push({base64 : '占位', code : '占位'})
            fetch('../static/cards.cdb')
            .then((response) => response.blob())
            .then(async (b) => {
                let result = await SQL.find(b);
                for (const i of result.values) {
                    let card = new Card(i as Array<string>);
                    getBase64(await exportImage(card))
                }
            });
    }
    onBeforeMount(load);

    function getBase64(obj) {
        return new Promise(function () {
            let reader = new FileReader();
            reader.readAsDataURL(obj.blob);
            reader.onload = function () {
                const a = reader.result as string
                const b = a.split(',')
                pics.push({base64 : b[b.length - 1], code : obj.code})
            };
        })
    }

</script>