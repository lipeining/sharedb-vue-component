<template>
    <div>
        <el-row>
            <el-col :span="18">
                <div id="toolbar-container">
                    <span class="ql-formats">
                        <button msg="undo" @click="editorUndo">
                            <i class="el-icon-d-arrow-left"></i>
                        </button>
                        <button msg="redo" @click="editorRedo">
                            <i class="el-icon-d-arrow-right"></i>
                        </button>
                    </span>
                    <span class="ql-formats">
                        <select class="ql-font"></select>
                        <select class="ql-size"></select>
                    </span>
                    <span class="ql-formats">
                        <button class="ql-bold"></button>
                        <button class="ql-italic"></button>
                        <button class="ql-underline"></button>
                        <button class="ql-strike"></button>
                    </span>
                    <span class="ql-formats">
                        <select class="ql-color"></select>
                        <select class="ql-background"></select>
                    </span>
                    <span class="ql-formats">
                        <button class="ql-script" value="sub"></button>
                        <button class="ql-script" value="super"></button>
                    </span>
                    <span class="ql-formats">
                        <button class="ql-header" value="1"></button>
                        <button class="ql-header" value="2"></button>
                        <button class="ql-blockquote"></button>
                        <button class="ql-code-block"></button>
                    </span>
                    <span class="ql-formats">
                        <button class="ql-list" value="ordered"></button>
                        <button class="ql-list" value="bullet"></button>
                        <button class="ql-indent" value="-1"></button>
                        <button class="ql-indent" value="+1"></button>
                    </span>
                    <span class="ql-formats">
                        <button class="ql-direction" value="rtl"></button>
                        <select class="ql-align"></select>
                    </span>
                    <span class="ql-formats">
                        <button class="ql-link"></button>
                        <button class="ql-image"></button>
                        <button class="ql-video"></button>
                        <button class="ql-formula"></button>
                    </span>
                    <span class="ql-formats">
                        <button class="ql-clean"></button>
                    </span>
                </div>
                <div v-loading="editorLock">
                    <!-- bidirectional data binding（双向数据绑定） -->
                    <quill-editor v-model="content" ref="myQuillEditor" :options="editorOption" @blur="onEditorBlur($event)" @focus="onEditorFocus($event)" @ready="onEditorReady($event)" @change="onEditorChange($event)">
                    </quill-editor>
                </div>
            </el-col>
        </el-row>
    </div>
</template>

<script>
    const ShareDB = require('sharedb/lib/client');
    const richText = require('rich-text');
    import ReconnectingWebSocket from 'reconnecting-websocket';
    ShareDB.types.register(richText.type);
    // Expose a singleton WebSocket connection to ShareDB server
    // import Quill from 'quill'
    // import Parchment from 'parchment'
    // const colors = ['red', 'blue', 'green', 'gold']
    // var userAttributor = new Parchment.Attributor.Attribute('user', 'class', {
    //   scope: Parchment.Scope.INLINE
    // })
    // Quill.register(userAttributor)
    export default {
        name: 'Doc',
        props: {
            'collectionName': String,
            'documentId': String,
            'socketUrl': String
        },
        data() {
            return {
                editorLock: true,
                editorOption: {
                    // some quill options
                    modules: {
                        // syntax: true,
                        toolbar: '#toolbar-container',
                        history: {
                            delay: 10000,
                            maxStack: 500,
                            userOnly: false,
                            // userOnly: true
                        }
                    },
                    placeholder: '',
                    theme: 'snow'
                },
                content: '',
                mydoc: ''
            }
        },
        computed: {
            editor() {
                return this.$refs.myQuillEditor.quill
            },
        },
        watch: {},
        mounted() {
            console.log('this is current quill instance object', this.editor);
            this.init();
        },
        created() {},
        methods: {
            enableEditor() {
                this.editor.enable();
                this.editorLock = false;
            },
            disableEditor() {
                this.editor.disable();
                this.editorLock = true;
            },
            init() {
                this.disableEditor();
                const socket = new ReconnectingWebSocket(this.socketUrl);
                const shareDBConnection = new ShareDB.Connection(socket);
                const collectionName = this.collectionName;
                const documentId = this.documentId;
                console.log(collectionName, documentId);
                try {
                    this.mydoc = shareDBConnection.get(collectionName, documentId);
                    this.mydoc.subscribe((err) => {
                        if (err) throw err;

                        if (!this.mydoc.type)
                            this.mydoc.create([], 'rich-text');
                        this.editor.setContents(this.mydoc.data);
                        this.enableEditor();
                        this.editor.on('text-change', (delta, oldDelta, source) => {
                            if (source !== 'user') return;
                            let d = new Date()
                            // 一般有多个操作，暂时在insert上加一个attributes
                            console.log(d.getSeconds() + JSON.stringify(delta));
                            this.mydoc.submitOp(delta);
                        })
                    });
                    this.mydoc.on('op', (op, source) => {
                        if (source) return;
                        console.log(op);
                        this.editor.updateContents(op)
                    });
                    this.mydoc.on('error', (err) => {
                        // console.log(err);
                        console.log(JSON.stringify(err));
                        console.log(err.message);
                        this.disableEditor();
                    });
                } catch (err) {
                    console.log(JSON.stringify(err));
                }
            },
            editorRedo() {
                console.log('redo')
                this.editor.history.redo();
            },
            editorUndo() {
                console.log('undo')
                this.editor.history.undo();
            },
            onEditorBlur(quill) {
                // console.log('editor blur!', quill)
            },
            onEditorFocus(quill) {
                // console.log('editor focus!', quill)
            },
            onEditorReady(quill) {
                // console.log('editor ready!', quill)
            },
            onEditorChange({ quill, html, text }) {

            }
        }
    }
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
    /* 
    button[class*='ql-'] {
    position: relative;
    transition: background-color 0.4s ease;
}

button[class*='ql-']::hover{
    background-color: rgba(255, 150, 0, 0.1) !important;
}

button[class*='ql-']::before,
button[class*='ql-']::after {
    z-index: 1;
    display: inline-block;
}
button[class*='ql-']:hover::before {
    content: '';
    position: absolute;
    top: 100%;
    left: 50%;
    transform: translate(-50%, 0);
    border: 5px solid transparent;
    border-bottom: 5px solid #444;
}

button[class*='ql-']:hover::after {
    content: attr(msg);
    position: absolute;
    top: 100%;
    left: 50%;
    font-size: 10px;
    transform: translate(-50%, 10px);
    padding: 10px;
    color: white;
    background-color: #444;
    border-radius: 4px;
    word-break: keep-all;
    line-height: 16px;
}
 */
</style>