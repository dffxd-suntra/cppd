<script>
import { VAceEditor } from "vue3-ace-editor";
import ace from "ace-builds";
import _, { head } from "lodash";
import { v4 as uuidv4 } from "uuid";

import cppModeUrl from "ace-builds/src-noconflict/mode-c_cpp?url";
import monokaiThemeUrl from "ace-builds/src-noconflict/theme-monokai?url";
ace.config.setModuleUrl("ace/mode/c_cpp", cppModeUrl);
ace.config.setModuleUrl("ace/theme/monokai", monokaiThemeUrl);

export default {
    components: {
    },
    data() {
        return {
            isNsfw: false,
            splitSymbool: [
                " ",
                "(",
                ")",
                "[",
                "]",
                ";",
                "{",
                "}",
                ",",
                "&&",
                "||",
                "->",
                "==",
                "!=",
                "<=",
                ">=",
                "*=",
                "/=",
                "+=",
                "-=",
                "%=",
                "<<=",
                ">>=",
                "&=",
                "^=",
                "|=",
                "++",
                "--",
                "<<",
                ">>",
                "=",
                "+",
                "-",
                "*",
                "/",
                "%",
                "&",
                "^",
                "|",
                "<",
                ">"
            ],
            inputContent: `#include <iostream>

using namespace std;

int main() {
    int a, b;
    cin >> a >> b;
    cout << a + b << endl;
    return 0;
}`,
            removeSpace: true,
            saveLine: 0,
            onlyPressing: false
        };
    },
    watch: {
        isNsfw(newValue, oldValue) {
            if (newValue) {
                document.title = "骗友队自动代码混淆";
            } else {
                document.title = "信友队自动代码混淆";
            }
        }
    },
    methods: {
        mixCode(code) {
            let that = this;
            function isLetter(symbool) {
                return ['a', 'b', 'c', 'd', 'e', 'f', 'g', 'h', 'i', 'j', 'k', 'l', 'm', 'n', 'o', 'p', 'q', 'r', 's', 't', 'u', 'v', 'w', 'x', 'y', 'z', 'A', 'B', 'C', 'D', 'E', 'F', 'G', 'H', 'I', 'J', 'K', 'L', 'M', 'N', 'O', 'P', 'Q', 'R', 'S', 'T', 'U', 'V', 'W', 'X', 'Y', 'Z', '1', '2', '3', '4', '5', '6', '7', '8', '9', '0'].includes(symbool);
            }
            function headerSplit(code) {
                let keys = ["include", "define", "error", "if", "elif", "else", "endif", "ifdef", "ifndef", "import", "line", "undef", "using", "pragma"];
                let header = "";
                for (let i in keys) {
                    let reg = new RegExp(`#\\s*${keys[i]}.*`, "g");
                    header += (code.match(reg) || []).join("\n");
                    code = code.replace(reg, "");
                    header = header.trim() + "\n";
                }
                return [header, code];
            }
            function formatCode(code) {
                let res = "";
                let cur = 0;
                while (cur < code.length) {
                    // 去空格
                    if (that.removeSpace && [" ", "\n", "\r", "\t"].includes(code[cur])) {
                        // 找到空格的头和尾
                        let prei = cur - 1;
                        while ([" ", "\n", "\r", "\t"].includes(code[cur])) {
                            cur++;
                        }
                        // 判断
                        // 检测 不能合并成新符号 例如 pair<int, pair<int, string> > 不能合并成 pair<int, <int, string>>
                        if (that.splitSymbool.includes(code[prei] + code[cur]) || (isLetter(code[prei]) && isLetter(code[cur]))) {
                            res += " ";
                        }
                        continue;
                    }
                    // 跳过双引号字符串
                    if (code[cur] == "\"") {
                        let backslashNum = 0;
                        while (cur < code.length) {
                            res += code[cur++];

                            if (code[cur - 1] == "\\") {
                                backslashNum++;
                            } else {
                                backslashNum = 0;
                            }
                            if (code[cur] == "\"" && backslashNum % 2 == 0) {
                                break;
                            }
                        }
                        res += code[cur++];
                        continue;
                    }
                    // 跳过单引号字符
                    if (code[cur] == "'") {
                        let backslashNum = 0;
                        while (cur < code.length) {
                            res += code[cur++];

                            if (code[cur - 1] == "\\") {
                                backslashNum++;
                            } else {
                                backslashNum = 0;
                            }
                            if (code[cur] == "'" && backslashNum % 2 == 0) {
                                break;
                            }
                        }
                        res += code[cur++];
                        continue;
                    }
                    // 去掉单行注释
                    if (code[cur] == "/" && code[cur + 1] == "/") {
                        cur += 2;
                        while (cur < code.length) {
                            if (code[cur] == '\n' || code[cur] == '\r') {
                                break;
                            }
                            cur++;
                        }
                        continue;
                    }
                    // 去掉多行注释
                    if (code[cur] == "/" && code[cur + 1] == "*") {
                        cur += 2;
                        while (cur < code.length) {
                            if (code[cur - 2] == '*' && code[cur - 1] == '/') {
                                break;
                            }
                            cur++;
                        }
                        continue;
                    }
                    // 普通字符加上就是了
                    res += code[cur++];
                }

                if (!that.removeSpace) {
                    res = res.replaceAll(/(\n|\r)/g, "");
                }
                return res;
            }
            function mixCode(code) {
                // 分割代码
                let spcode = [];
                let orii = -Infinity;
                while (code.length != "") {
                    // 双引号分割
                    if (code[0] == "\"") {
                        let len = 0;
                        let backslashNum = 0;
                        while (len < code.length) {
                            len++;

                            if (code[len - 1] == "\\") {
                                backslashNum++;
                            } else {
                                backslashNum = 0;
                            }
                            if (code[len] == "\"" && backslashNum % 2 == 0) {
                                break;
                            }
                        }
                        len++;
                        spcode.push(code.substring(0, len));
                        code = code.substring(len);
                        continue;
                    }

                    // 单引号分割
                    if (code[0] == "'") {
                        let len = 0;
                        let backslashNum = 0;
                        while (len < code.length) {
                            len++;

                            if (code[len - 1] == "\\") {
                                backslashNum++;
                            } else {
                                backslashNum = 0;
                            }
                            if (code[len] == "'" && backslashNum % 2 == 0) {
                                break;
                            }
                        }
                        len++;
                        spcode.push(code.substring(0, len));
                        code = code.substring(len);
                        continue;
                    }

                    // 符号分割
                    let has = false;
                    for (let i in that.splitSymbool) {
                        if (code.substring(0, that.splitSymbool[i].length) == that.splitSymbool[i]) {
                            spcode.push(code.substring(0, that.splitSymbool[i].length));
                            code = code.substring(that.splitSymbool[i].length);
                            has = true;
                            break;
                        }
                    }
                    if (has) {
                        continue;
                    }

                    // 普通字符分割
                    if (spcode.length - 1 > orii) {
                        spcode.push("");
                        orii = spcode.length - 1;
                    }
                    spcode[orii] += code[0];
                    code = code.substring(1);
                }

                // 合并代码
                /* let getTag = (function () {
                    let tagHash = new Map();
                    let tagLen = 10;
                    let tagLenMax = (1 << tagLen) - 1;
                    return function () {
                        let i;
                        do {
                            i = _.random(0, tagLenMax);
                        } while (tagHash.get(i));
                        tagHash.set(i, true);

                        let t = tagLen;
                        let res = "";
                        while (t--) {
                            res = "" + ["0", "8"][i & 1] + res;
                            i >>= 1;
                        }
                        return "D" + res;
                    }
                })(); */
                let getTag = (function () {
                    let tagHash = new Map();
                    return function () {
                        let id;
                        do {
                            id = uuidv4();
                        } while (tagHash.get(id));
                        tagHash.set(id, true);
                        return "UUID_" + id.replaceAll("-", "");
                    }
                })();

                let res = "";
                let tagPool = [];
                let valuePool = _.cloneDeep(spcode);
                let newValuePool = [];
                let lastTag;

                while (valuePool.length > 1) {
                    while (valuePool.length > 0) {
                        let tag = getTag();
                        tagPool.push([tag, valuePool.splice(0, _.random(2, 5)).join("").trim()]);
                        newValuePool.push(tag + " ");
                    }
                    valuePool = newValuePool;
                    newValuePool = [];

                    tagPool = _.shuffle(tagPool);
                    res += tagPool.map(element => `#define ${element[0]} ${element[1]}`).join("\n") + "\n";
                    if (valuePool.length == 1) {
                        lastTag = valuePool[0].trim();
                    }
                    tagPool = [];
                }
                let enterTag = getTag();
                let enterValue = "_".repeat(8) + (that.isNsfw ? "PYD" : "XYD") + "_".repeat(8);
                res += `#define ${enterTag}(${enterValue}) ${enterValue} \n`;
                res += `${enterTag}(${lastTag})`;

                return res;
            }

            code = code.trim();
            if (code == "") {
                return "";
            }

            // 提取头文件
            let header;
            [header, code] = headerSplit(code);
            code = formatCode(code);
            if (this.onlyPressing) {
                return header + code;
            }
            code = mixCode(code);
            return header + code;
        }
    },
    computed: {
        outputContent() {
            let code = this.inputContent.split("\n");
            let pre = code.splice(0, this.saveLine);
            return pre.join("\n") + this.mixCode(code.join("\n"));
        }
    },
    components: {
        VAceEditor,
    },
    mounted() {
    }
};
</script>

<template>
    <el-container id="pageContainer">
        <el-main id="main">
            <h1 style="margin: 0;" @click="isNsfw = !isNsfw"><img src="./assets/xyd_logo.png" id="xyd_logo" v-show="!isNsfw" /><img src="./assets/pyd_logo.png" id="pyd_logo" v-show="isNsfw" />&nbsp;C++&nbsp;代码混淆</h1>
            <el-text size="large">对头文件有处理,混淆出了问题请自行处理</el-text>

            <el-row style="margin-top: 1rem;">
                <el-col :span="24">
                    <el-text size="large">处理前:</el-text>
                    <VAceEditor v-model:value="inputContent" lang="c_cpp" theme="monokai" :max-lines="Infinity" :min-lines="30"></VAceEditor>
                </el-col>
                <el-col :span="24">
                    <el-form>
                        <el-form-item>
                            <h3>参数:</h3>
                        </el-form-item>
                        <el-form-item label="保留行数"><el-input-number v-model="saveLine" :min="0" :step="1" step-strictly /></el-form-item>
                        <el-form-item label=""><el-checkbox v-model="removeSpace" label="去掉空格" size="large" /></el-form-item>
                        <el-form-item label=""><el-checkbox v-model="onlyPressing" label="仅压行" size="large" /></el-form-item>
                    </el-form>
                    <el-text size="large">处理后:</el-text>
                    <VAceEditor v-model:value="outputContent" :readonly="true" lang="c_cpp" theme="monokai" :max-lines="Infinity" :min-lines="30"></VAceEditor>
                </el-col>
            </el-row>

            <h3 style="margin-bottom: 0;">感谢 <a href="https://www.eee.dog/tech/rand-pic-api.html">EEE.Dog</a> 的随机背景图api</h3>
        </el-main>
    </el-container>
    <div id="animeBackgroundImage"></div>
</template>

<style scoped>
#pageContainer {
    overflow: hidden;
    border-radius: 0.5rem;
    margin-top: 2rem;
    margin-bottom: calc(100vh + 2rem);
    max-width: 1000px;
    background-color: rgba(240, 225, 161, 60%);
    backdrop-filter: blur(5px);
    box-shadow: 0 0 10px 1px rgba(0, 0, 0, 20%);
}

#main {
    width: 100%;
}

#inputEditor,
#outputEditor {
    width: 100%;
}

#xyd_logo,
#pyd_logo {
    vertical-align: middle;
    height: 1.5em;
}

.ace_editor {
    background-color: rgba(39, 40, 34, 60%);
}

.el-form-item {
    margin: 0;
}
</style>
