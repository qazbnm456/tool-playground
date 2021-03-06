<template>
    <el-row type="flex" class="row-bg" justify="center">
        <el-col :lg="12" :md="14" :sm="18" :xs="22">
            <el-row type="flex" class="row-bg" justify="center">
                <el-col :span="24">
                    <el-card class="box-card row-bg">
                        <el-row slot="header" type="flex" class="clearfix" justify="center" :gutter="20">
                            <el-col :span="20">
                                <el-input
                                        type="text"
                                        placeholder="明文"
                                        v-model="plaintext">
                                </el-input>
                            </el-col>
                            <el-col :span="4">
                                <el-input
                                        type="text"
                                        placeholder="Salt"
                                        v-model="salt">
                                </el-input>
                            </el-col>
                        </el-row>
                        <div class="text item" v-for="item in hashed">
                            <el-row slot="header" type="flex" class="clearfix" justify="center" :gutter="20">
                                <el-col :span="4">{{ item.label }}</el-col>
                                <el-col :span="20"><code>{{ item.action() }}</code></el-col>
                            </el-row>
                        </div>
                        <div class="text item" v-if="tags.length">
                            <el-row slot="header" type="flex" class="clearfix" justify="center" :gutter="20">
                                <el-col :span="4">自定义</el-col>
                                <el-col :span="20"><code>{{ customResult }}</code></el-col>
                            </el-row>
                        </div>
                    </el-card>
                    <el-card class="box-card">
                        <div slot="header" type="flex" class="clearfix" justify="center">自定义哈希</div>
                        <div class="text item">
                            <el-button-group>
                                <el-button type="default" size="small" @click="push('md5(')">MD5</el-button>
                                <el-button type="default" size="small" @click="push('sha1(')">SHA1</el-button>
                                <el-button type="default" size="small" @click="push('sha256(')">SHA256</el-button>
                                <el-button type="default" size="small" @click="push('sha512(')">SHA512</el-button>
                            </el-button-group>
                            <el-button-group>
                                <el-button type="primary" size="small" @click="push('plaintext')">PLAINTEXT</el-button>
                                <el-button type="primary" size="small" @click="push('salt')">SALT</el-button>
                            </el-button-group>
                            <el-button-group>
                                <el-button type="default" size="small" @click="push(')')">)</el-button>
                                <el-button type="default" size="small" @click="tags=[]">清空</el-button>
                            </el-button-group>
                        </div>
                        <div class="text item" v-if="tags.length">
                            <el-tag
                                    :key="tag.name"
                                    v-for="tag in tags"
                                    :closable="true"
                                    :type="tag.type"
                                    @close="pop(tag)"
                                    class="tag"
                            >{{ tag.name }}</el-tag>
                        </div>
                        <div class="text item" v-if="tags.length">
                            Code: <code>{{ customHashCode }}</code>
                        </div>
                    </el-card>
                </el-col>
            </el-row>
        </el-col>
    </el-row>
</template>

<script>
import CryptoJS from "crypto-js"

const hashTable = {
    md5: (val) => CryptoJS.MD5(val).toString(),
    sha1: (val) => CryptoJS.SHA1(val).toString(),
    sha256: (val) => CryptoJS.SHA256(val).toString(),
    sha512: (val) => CryptoJS.SHA512(val).toString(),
}

export default {
    name: 'hashed',
    data () {
        return {
            plaintext: "",
            salt: "",
            tags: [],
            hashChain: [],
            customHashCode: ''
        }
    },
    methods: {
        push (m) {
            this.tags.push({
                name: m,
                type: 'gray'
            })
        },
        pop (tag) {
            this.tags.splice(this.tags.indexOf(tag), 1);
        }
    },
    watch: {
        tags (val, oldVal) {
            this.hashChain = []
            for (let item of this.tags) {
                this.hashChain.push(item.name)
            }
        }
    },
    computed: {
        customResult () {
            let code = [];
            for (let hash of this.hashChain) {
                if (hash.substring(0, hash.length-1) in hashTable) {
                    code.push(`hashTable.${hash}`)
                } else if (['plaintext', 'salt'].indexOf(hash) >= 0) {
                    code.push(`this.${hash}`)
                } else {
                    code.push(hash)
                }
            }

            code = code.join('').replace(/\s/g, '')
            code = code.replace(/plaintext(?![\(\)])/g, 'plaintext+')
                    .replace(/salt(?![\(\)])/g, 'salt+')
                    .replace(/\)(?=[a-z0-9])/ig, ')+')
            this.customHashCode = code

            try {
                return this.magic(eval(code))
            } catch (e) {return "自定义表达式错误"}
        },
        hashed () {
            return [
                {
                    label: "MD5",
                    action: () => this.magic(hashTable.md5(`${this.plaintext}${this.salt}`))
                },
                {
                    label: "SHA1",
                    action: () => this.magic(hashTable.sha1(`${this.plaintext}${this.salt}`))
                },
                {
                    label: "SHA256",
                    action: () => this.magic(hashTable.sha256(`${this.plaintext}${this.salt}`))
                },
                {
                    label: "SHA512",
                    action: () => this.magic(hashTable.sha512(`${this.plaintext}${this.salt}`))
                },
                {
                    label: "RIPEMD160",
                    action: () => this.magic(CryptoJS.RIPEMD160(`${this.plaintext}${this.salt}`).toString())
                },
                {
                    label: "HMAC(MD5)",
                    action: () => this.magic(CryptoJS.HmacMD5(this.plaintext, this.salt).toString())
                },
                {
                    label: "HMAC(SHA1)",
                    action: () => this.magic(CryptoJS.HmacSHA1(this.plaintext, this.salt).toString())
                },
                {
                    label: "HMAC(SHA256)",
                    action: () => this.magic(CryptoJS.HmacSHA256(this.plaintext, this.salt).toString())
                },
                {
                    label: "HMAC(SHA512)",
                    action: () => this.magic(CryptoJS.HmacSHA512(this.plaintext, this.salt).toString())
                }
            ]
        },
    },
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.row-bg {
    margin-bottom: 30px;
}
.text {
    font-size: 14px;
}

.item {
    padding: 18px 0;
}

.clearfix:before,
.clearfix:after {
    display: table;
    content: "";
}
.clearfix:after {
    clear: both
}
.box-card {
    word-wrap: break-word;
    word-break: break-all;
}
.tag {
    margin: 0 10px 5px 0;
}
</style>
