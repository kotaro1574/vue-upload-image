<template>
    <div>
        <p>タイトル：<input type="text" v-model="title" /></p>
        <p><input type="file" @change="confirmImage()" ref="preview" v-if="view" /></p>

        <!-- 確認用画像 -->
        <p v-if="confirmedImage">
            <img class="img" :src="confirmedImage" />
        </p>

        <p>{{ message }}</p>

        <p>
            <button @click="uploadImage">アップロード</button>
        </p>

        <!-- 画像一覧 -->
        <table border="1">
            <tr>
                <th>title</th>
                <th>画像</th>
            </tr>
            <tr v-for="image in images" :key="image.id">
                <td>{{ image.title }}</td>
                <td><img class="img" :src="image.path" /></td>
            </tr>
        </table>
    </div>
</template>

<script>
import axios from "axios";
export default {
    data() {
        return {
            message: "",
            file: "",
            title: "",
            view: true,
            images: {},
            confirmedImage: ""
        };
    },
    created: function() {
        this.getImage();
        console.log(this.images);
    },
    methods: {
        getImage() {
            axios
                .get("http://127.0.0.1:8000/api/images/")
                .then(response => {
                    this.images = response.data;
                })
                .catch(err => {
                    this.message = err;
                });
        },
        confirmImage() {
            // this.message = "";
            // this.file = target.files[0];
            this.file = this.$refs.preview.files[0]
            // if (!this.file.type.match("image.*")) {
            //     this.message = "画像ファイルを選択して下さい";
            //     this.confirmedImage = "";
            //     return;
            // }
            this.createImage(this.file);
        },
        createImage(file) {
            let reader = new FileReader();
            reader.readAsDataURL(file);
            reader.onload = e => {
                this.confirmedImage = e.target.result;
            };
        },
        uploadImage() {
            let data = new FormData();
            data.append("file", this.file);
            data.append("title", this.title);

            axios
                .post("http://127.0.0.1:8000/api/images/", data)
                .then(response => {
                    this.getImage();
                    this.message = response.data.success;
                    this.confirmedImage = "";
                    this.title = "";
                    this.file = "";

                    //ファイルを選択のクリア
                    this.view = false;
                    this.$nextTick(function() {
                        this.view = true;
                    });
                })
                .catch(err => {
                    this.message = err.response.data.errors;
                });
        }
    }
};
</script>

<style>
.img {
    width: 100px;
}
</style>
