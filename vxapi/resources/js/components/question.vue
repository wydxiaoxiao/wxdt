<template>
    <el-container>
        <el-header class="header">
            <el-tabs v-model="activeName" tab-position='top' stretch @tab-click="handleClick">
                <el-tab-pane label="选择题" name="choose"></el-tab-pane>
                <el-tab-pane label="多选题" name="mutichoose"></el-tab-pane>
                <el-tab-pane label="判断题" name="judge"></el-tab-pane>
                <el-tab-pane label="填空题" name="fillBlank"></el-tab-pane>
            </el-tabs>
        </el-header>
        <el-main class="main">
            <el-table :data="tableData" max-height="500px" stripe border>
                <el-table-column fixed prop="id" label="题号" width="100%"></el-table-column>
                <el-table-column prop="question" label="题目" width="600%">
                    <template slot-scope="scope">
                        <span v-if="scope.row.isUpdate">
                            <el-input v-model="scope.row.question" placeholder="请输入内容"></el-input>
                        </span>
                        <span v-else>{{ scope.row.question }}</span>
                    </template>
                </el-table-column>
                <el-table-column v-if="activeName=='choose'||activeName=='mutichoose'" prop="a" label="A选项"
                                 width="200%">
                    <template slot-scope="scope">
                        <span v-if="scope.row.isUpdate">
                            <el-input v-model="scope.row.a" placeholder="请输入内容"></el-input>
                        </span>
                        <span v-else>{{ scope.row.a }}</span>
                    </template>
                </el-table-column>
                <el-table-column v-if="activeName=='choose'||activeName=='mutichoose'" prop="b" label="B选项"
                                 width="200%">
                    <template slot-scope="scope">
                        <span v-if="scope.row.isUpdate">
                            <el-input v-model="scope.row.b" placeholder="请输入内容"></el-input>
                        </span>
                        <span v-else>{{ scope.row.b }}</span>
                    </template>
                </el-table-column>
                <el-table-column v-if="activeName=='choose'||activeName=='mutichoose'" prop="c" label="C选项"
                                 width="200%">
                    <template slot-scope="scope">
                        <span v-if="scope.row.isUpdate">
                            <el-input v-model="scope.row.c" placeholder="请输入内容"></el-input>
                        </span>
                        <span v-else>{{ scope.row.c }}</span>
                    </template>
                </el-table-column>
                <el-table-column v-if="activeName=='choose'||activeName=='mutichoose'" prop="d" label="D选项"
                                 width="200%">
                    <template slot-scope="scope">
                        <span v-if="scope.row.isUpdate">
                            <el-input v-model="scope.row.d" placeholder="请输入内容"></el-input>
                        </span>
                        <span v-else>{{ scope.row.d }}</span>
                    </template>
                </el-table-column>

                <el-table-column prop="answer" label="答案" width="400%">
                    <template slot-scope="scope">
                        <span v-if="scope.row.isUpdate">
                            <el-input v-model="scope.row.answer" placeholder="请输入内容"></el-input>
                        </span>
                        <span v-else>{{ scope.row.answer }}</span>
                    </template>
                </el-table-column>

                <el-table-column fixed="right" label="操作" width="110%">
                    <template slot-scope="scope">
                        <span v-if="scope.row.isUpdate">
                            <el-button @click="saveRow(scope.$index, scope.row)" type="text">保存</el-button>
                        </span>
                        <span v-else>
                            <el-button @click="updateRow(scope.$index, scope.row)" type="text">修改</el-button>
                        </span>
                        <el-button @click.native.prevent="deleteRow(scope.$index, scope.row, tableData)" type="text">
                            移除
                        </el-button>
                    </template>
                </el-table-column>
            </el-table>
        </el-main>
        <el-footer>
            <el-row>
                <el-col :offset="19" :span="2">
                    <el-button @click="drawer = true" type="primary" round>新增题目</el-button>
                </el-col>
            </el-row>
        </el-footer>

        <el-drawer v-if="questionNum == 0" :visible.sync="drawer" :direction="direction" :before-close="handleClose"
                   :with-header="false">
            <el-row class="row" type="flex" align="middle">
                <el-col :offset="6" :span="8">
                    <div>
                        <el-button class="button" type="primary" @click="buttonClick(1)">单题添加</el-button>
                    </div>
                </el-col>
                <el-col :span="8">
                    <div>
                        <el-button class="button" type="success" @click="buttonClick(-1)">批量添加</el-button>
                    </div>
                </el-col>
            </el-row>
        </el-drawer>

        <el-drawer v-if="questionNum == 1" :visible.sync="drawer" :direction="direction" :before-close="handleClose"
                   :with-header="false">
            <el-row class="row-up" type="flex" align="bottom" :gutter="5">
                <el-form :model="newQuestion" :rules="rules" ref="questionForm">
                    <el-col :span="3">
                        <el-input v-model="activeName" disabled></el-input>
                    </el-col>
                    <el-col :span="9">
                        <el-form-item prop="question">
                            <el-input v-model="newQuestion.question" placeholder="请输入题目"></el-input>
                        </el-form-item>
                    </el-col>
                    <el-col v-if="activeName=='choose'||activeName=='mutichoose'" :span="2">
                        <el-input v-model="newQuestion.options.a" placeholder="A"></el-input>
                    </el-col>
                    <el-col v-if="activeName=='choose'||activeName=='mutichoose'" :span="2">
                        <el-input v-model="newQuestion.options.b" placeholder="B"></el-input>
                    </el-col>
                    <el-col v-if="activeName=='choose'||activeName=='mutichoose'" :span="2">
                        <el-input v-model="newQuestion.options.c" placeholder="C"></el-input>
                    </el-col>
                    <el-col v-if="activeName=='choose'||activeName=='mutichoose'" :span="2">
                        <el-input v-model="newQuestion.options.d" placeholder="D"></el-input>
                    </el-col>
                    <el-col :span="4">
                        <el-form-item prop="answer">
                            <span v-if="activeName=='choose'">
                                <el-select v-model="newQuestion.answer" placeholder="请选择答案">
                                    <el-option value="a"></el-option>
                                    <el-option value="b"></el-option>
                                    <el-option value="c"></el-option>
                                    <el-option value="d"></el-option>
                                </el-select>
                            </span>
                            <span v-else-if="activeName=='judge'">
                                <el-select v-model="newQuestion.answer" placeholder="请选择答案">
                                    <el-option value="0"></el-option>
                                    <el-option value="1"></el-option>
                                </el-select>
                            </span>
                            <span v-else>
                                <el-input v-model="newQuestion.answer" placeholder="请输入答案"></el-input>
                            </span>
                        </el-form-item>
                    </el-col>
                </el-form>
            </el-row>
            <el-row type="flex">
                <el-col :offset="20" :span="3">
                    <el-button type="primary" @click="insertRow('questionForm')" round>新增题目</el-button>
                </el-col>
            </el-row>
        </el-drawer>
        <el-drawer v-if="questionNum == -1" :visible.sync="drawer" :direction="direction" :before-close="handleClose"
                   :with-header="false">
            <el-row class="row-middle" type="flex" align="top">
                <el-col :offset="4" :span="16">
                    <el-upload class="upload" ref="upload" :action="url" name="question"
                               :file-list="fileList"
                               :on-success="filesuccess" :on-error="fileerror" :auto-upload="false" :limit=1
                               :on-exceed="fileexceed">
                        <el-button slot="trigger" type="primary">选择文件</el-button>
                        <el-button style="margin-left: 10px;" type="success" @click="submitUpload">上传</el-button>
                        <el-link style="margin-left: 10px;" type="danger" :href="url+'csv'">获取样例</el-link>
                        <div slot="tip" class="el-upload__tip">只能上传csv或txt文件</div>
                    </el-upload>
                </el-col>
            </el-row>
        </el-drawer>
    </el-container>
</template>

<script>
    export default {
        data() {
            return {
                url: 'http://wxdt.dutbit.com/control/',
                // url: 'http://127.0.0.1:8000/control/',
                activeName: 'choose',
                tableData: [],
                drawer: false,
                direction: "ttb",
                questionNum: 0,
                newQuestion: {options: {}},
                row: [],
                rules: {
                    question: [
                        {required: true, message: '请输入题目', trigger: 'blur'}
                    ],
                    answer: [
                        {required: true, message: '请输入答案', trigger: 'blur'}
                    ]
                },
                fileList: [],
            };
        },
        mounted: function () {
            this.handleClick();
        },
        methods: {
            handleClick() {
                var self = this;
                var url = this.url + this.activeName;
                axios.get(url)
                    .then(function (response) {
                        self.tableData = response.data;
                    })
                    .catch(function (error) {
                        alert("网络错误！");
                        console.log(error);
                    });
            },
            handleClose(done) {
                var self = this;
                this.$confirm('确认关闭？')
                    .then(_ => {
                        self.questionNum = 0;
                        self.newQuestion = {options: {}};
                        done();
                    })
                    .catch(_ => {
                        return
                    });
            },
            buttonClick(number) {
                this.questionNum = number;
            },
            insertRow(formName) {
                var self = this;
                var url = this.url + this.activeName;
                this.$refs[formName].validate((valid) => {
                    if (valid) {
                        if (this.activeName == 'judge') {
                            self.newQuestion.options = ["错", "对"];
                        }
                        axios.post(url, JSON.stringify(self.newQuestion))
                            .then(function (response) {
                                if (response.data == 1) {
                                    alert('添加成功！');
                                    window.location.reload();
                                }
                            })
                            .catch(function (error) {
                                alert("网络错误！");
                                console.log(error);
                            });
                    }
                });
            },
            updateRow(index, row) {
                this.$set(row, 'isUpdate', true);
            },
            saveRow(index, row) {
                var url = this.url + this.activeName;
                var data_m = Object.assign({}, this.tableData[index]);
                delete data_m.isUpdate;
                var senddata = {options: {}};
                senddata.id = data_m.id;
                delete data_m.id;
                senddata.question = data_m.question;
                delete data_m.question;
                senddata.answer = data_m.answer;
                delete data_m.answer;
                for (var i in data_m) {
                    senddata.options[i] = data_m[i];
                }
                axios.put(url, JSON.stringify(senddata))
                    .then(function (response) {
                    })
                    .catch(function (error) {
                        alert("网络错误！");
                        console.log(error);
                    });
                this.$set(row, 'isUpdate', false);
            },
            deleteRow(index, row, tabledata) {
                var url = this.url + this.activeName;
                this.$confirm('确认删除？')
                    .then(_ => {
                        this.$set(row, 'isUpdate', false);
                        axios.delete(url, {data: row.id})
                            .then(function (response) {
                                if (response.data == row.id)
                                    tabledata.splice(index, 1);
                            })
                            .catch(function (error) {
                                alert("网络错误！");
                                console.log(error);
                            });
                    })
                    .catch(_ => {
                        return
                    });
            },
            filesuccess() {
                alert('添加成功！');
                window.location.reload();
            },
            fileerror() {
                alert('网络错误！');
            },
            submitUpload() {
                this.$refs.upload.submit();
            },
            fileexceed() {
                alert('一次只能上传一个文件');
            },
        }
    }
    ;
</script>

<style scoped>
    .header {
        margin: 0px 20% 0px 10% !important;
    }

    .main {
        margin: 0px 9% 1% 0% !important;
    }

    .row {
        margin: 0 2% !important;
        height: 90%;
    }

    .row-up {
        margin: 0 2% !important;
        height: 55%;
    }

    .row-middle {
        height: 100%;
    }

    .upload {
        margin-top: 5%;
    }

    .el-form {
        width: 100% !important;
    }

    .button {
        width: 50% !important;
    }
</style>
