<template>
    <div style="margin: 0px 0px 15px 0px;font-size: 14px;">
        <el-row class="header">
            <el-col :lg="12" :md="12" :sm="12" :xs="24">
                <div class="select-row">
                    <span>
                        <el-select :value="this.language" class="left-adjust" size="small" @change="onLangChange">
                            <el-option v-for="item in languages" :key="item" :value="item">{{ item }}
                            </el-option>
                        </el-select>
                    </span>
                    <!--重置代码-->
                    <span>
                        <el-tooltip :content="$t('m.Reset_Code')" placement="top">
                            <el-button icon="el-icon-refresh" size="small" @click="onResetClick"></el-button>
                        </el-tooltip>
                    </span>
                    <!--上传文件-->
                    <span>
                        <el-tooltip :content="$t('m.Upload_file')" placement="top">
                            <el-button icon="el-icon-upload" size="small" @click="onUploadFile"></el-button>
                        </el-tooltip>
                    </span>
                    <span>
                        <input id="file-uploader" style="display: none" type="file" @change="onUploadFileDone" />
                    </span>
                </div>
            </el-col>
            <el-col :lg="12" :md="12" :sm="12" :xs="24">
                <div class="select-row fl-right">
                    <span>{{ $t('m.Display_Mode') }}:</span>
                    <span>
                        <el-tooltip :content="$t('m.Switch_Display')" placement="top">
                            <el-button icon="el-icon-exchange" size="small" @click="onSwitchDisplay"></el-button>
                        </el-tooltip>
                    </span>
                    <span>{{ $t('m.Theme') }}:</span>
                    <el-select :value="this.theme" class="right-adjust" size="small" @change="onThemeChange">
                        <el-option v-for="item in themes" :key="item.label" :label="$t('m.' + item.label)"
                            :value="item.value">{{ $t('m.' + item.label) }}
                        </el-option>
                    </el-select>
                </div>
            </el-col>
        </el-row>
        <codemirror ref="myEditor" :options="options" :value="value" @change="onEditorCodeChange">
        </codemirror>
    </div>
</template>
<script>
import utils from '@/common/utils';
import { codemirror, CodeMirror } from 'vue-codemirror-lite';

// 风格对应的样式
import 'codemirror/theme/monokai.css';
import 'codemirror/theme/solarized.css';
import 'codemirror/theme/material.css';

// highlightSelectionMatches
import 'codemirror/addon/scroll/annotatescrollbar.js';
import 'codemirror/addon/search/matchesonscrollbar.js';
import 'codemirror/addon/dialog/dialog.js';
import 'codemirror/addon/dialog/dialog.css';
import 'codemirror/addon/search/searchcursor.js';
import 'codemirror/addon/search/search.js';
import 'codemirror/addon/search/match-highlighter.js';

// mode
import 'codemirror/mode/clike/clike.js';
import 'codemirror/mode/python/python.js';
import 'codemirror/mode/pascal/pascal.js'; //pascal
import 'codemirror/mode/go/go.js'; //go
import 'codemirror/mode/d/d.js'; //d
import 'codemirror/mode/haskell/haskell.js'; //haskell
import 'codemirror/mode/mllike/mllike.js'; //OCaml
import 'codemirror/mode/perl/perl.js'; //perl
import 'codemirror/mode/php/php.js'; //php
import 'codemirror/mode/ruby/ruby.js'; //ruby
import 'codemirror/mode/rust/rust.js'; //rust
import 'codemirror/mode/javascript/javascript.js'; //javascript
import 'codemirror/mode/fortran/fortran.js'; //fortran
// active-line.js
import 'codemirror/addon/selection/active-line.js';

// foldGutter
import 'codemirror/addon/fold/foldgutter.css';
import 'codemirror/addon/fold/foldgutter.js';

import 'codemirror/addon/edit/matchbrackets.js';
import 'codemirror/addon/edit/matchtags.js';
import 'codemirror/addon/edit/closetag.js';
import 'codemirror/addon/edit/closebrackets.js';
import 'codemirror/addon/fold/brace-fold.js';
import 'codemirror/addon/fold/indent-fold.js';
import 'codemirror/addon/hint/show-hint.css';
import 'codemirror/addon/hint/show-hint.js';
import 'codemirror/addon/hint/anyword-hint.js';
import 'codemirror/addon/selection/mark-selection.js';

export default {
    name: 'CodeMirror',
    components: {
        codemirror,
    },
    props: {
        value: {
            type: String,
            default: '',
        },
        languages: {
            type: Array,
            default: () => {
                return ['C', 'C++', 'Java17', 'Python3', 'Python2'];
            },
        },
        language: {
            type: String,
            default: 'C++',
        },
        theme: {
            type: String,
            default: 'solarized',
        },
    },
    data() {
        return {
            options: {
                // codemirror options
                tabSize: 4,
                mode: 'text/x-csrc',
                theme: 'solarized',
                // 显示行号
                lineNumbers: true,
                line: true,
                // 代码折叠
                foldGutter: true,
                gutters: ['CodeMirror-linenumbers', 'CodeMirror-foldgutter'],
                lineWrapping: true,
                // 选中文本自动高亮，及高亮方式
                styleSelectedText: true,
                showCursorWhenSelecting: true,
                highlightSelectionMatches: { showToken: /\w/, annotateScrollbar: true },
                matchBrackets: true, //括号匹配
                indentUnit: 4, //一个块（编辑语言中的含义）应缩进多少个空格
                styleActiveLine: true,
                autoCloseBrackets: true,
                autoCloseTags: true,
                hintOptions: {
                    // 当匹配只有一项的时候是否自动补全
                    completeSingle: false,
                },
            },
            mode: {
                C: 'text/x-csrc',
            },
            themes: [
                { label: 'monokai', value: 'monokai' },
                { label: 'solarized', value: 'solarized' },
                { label: 'material', value: 'material' },
            ],
        };
    },
    mounted() {
        utils.getLanguages().then((languages) => {
            let mode = {};
            languages.forEach((lang) => {
                mode[lang.name] = lang.contentType;
            });
            this.mode = mode;
            this.editor.setOption('mode', this.mode[this.language]);
        });
        this.editor.on('inputRead', (instance, changeObj) => {
            if (/\w|\./g.test(changeObj.text[0]) && changeObj.origin !== 'complete') {
                instance.showHint({
                    hint: CodeMirror.hint.anyword,
                    completeSingle: false,
                    range: 1000, // 附近多少行代码匹配
                });
            }
        });
    },
    methods: {
        onSwitchDisplay() {
            this.$emit('switchDisplayMode');
        },
        onEditorCodeChange(newCode) {
            this.$emit('update:value', newCode);
        },
        onLangChange(newVal) {
            this.editor.setOption('mode', this.mode[newVal]);
            this.$emit('changeLang', newVal);
        },
        onThemeChange(newTheme) {
            this.editor.setOption('theme', newTheme);
            this.$emit('changeTheme', newTheme);
        },
        onResetClick() {
            this.$emit('resetCode');
        },
        onUploadFile() {
            document.getElementById('file-uploader').click();
        },
        onUploadFileDone() {
            let f = document.getElementById('file-uploader').files[0];
            let fileReader = new window.FileReader();
            let self = this;
            fileReader.onload = function (e) {
                var text = e.target.result;
                self.editor.setValue(text);
                document.getElementById('file-uploader').value = '';
            };
            fileReader.readAsText(f, 'UTF-8');
        },
    },
    computed: {
        editor() {
            // get current editor object
            return this.$refs.myEditor.editor;
        },
        currentLanguage() {
            return this.language;
        },
    },
    watch: {
        theme(newVal, oldVal) {
            this.editor.setOption('theme', newVal);
        },
    },
};
</script>

<style scoped>
.header {
    margin-bottom: 10px;
    margin-right: 5px;
    margin-left: 5px;
}

.header .left-adjust {
    width: 170px;
    margin-left: 5px;
}

.header .right-adjust {
    width: 140px;
    margin-left: 5px;
}

.select-row {
    margin-top: 4px;
}

@media screen and (max-width: 768px) {
    .select-row span {
        margin-right: 2px;
    }
}

@media screen and (min-width: 768px) {
    .select-row span {
        margin-right: 3px;
    }

    .fl-right {
        float: right;
    }
}
</style>

<style>
.el-icon-exchange {
    background: url('exchange.svg') center no-repeat;
    font-size: 12px;
    background-size: cover;
}

.el-icon-exchange:before {
    content: "替";
    font-size: 12px;
    visibility: hidden;
}

.CodeMirror {
    height: 600px !important;
}

.CodeMirror-scroll {
    min-height: 549px;
    max-height: 1000px;
}

.cm-s-monokai .cm-matchhighlight {
    background-color: rgba(73, 72, 62, 0.99);
}

.cm-s-solarized .cm-matchhighlight {
    background-color: #d7d4f0;
}

.cm-s-material .cm-matchhighlight {
    background-color: rgba(128, 203, 196, 0.2);
}
</style>
