<template>
    <div class="setting-level">
        <h2 class="h2">等级设置</h2>
        <div class="setting-level">
            <Row  class="func-card-container" >
                <i-col span="12"  class="member-level">
                    <div class="func-card">
                        <div class="card-title"><span>会员等级模式</span></div>
                        <Row  class=" big-block-narrow">
                            <i-col span="10"  class="func-row">
                                <div class="property-name">模式状态</div>
                                <div v-if="activeLevelMode" class="property-value">
                                    <div class="status-icon"></div>
                                    <div class="status-text">已启用</div>
                                    <div @click="openDisablePop"
                                         class="status-close action-text">停用</div>
                                </div>
                                <div v-else class="property-value">
                                    <div class="status-icon-disable"></div>
                                    <div class="status-text">已停用</div>
                                    <div @click="openEnablePop" class="status-close action-text">启用</div>
                                </div>
                            </i-col>

                            <i-col span="10"  class="func-row">
                                <div class="property-name">等级有效期</div>
                                <div class="property-value">
                                    <div class="normal-status-text">
                                        <span>{{ grade.gradeExpirationYear }} 个自然年</span>
                                        <Poptip  trigger="click" placement="top" :transfer="true"
                                                 v-model="gradeBubble"
                                        >
                                            <span><i  class="icon iconfont iconEdit"  title="编辑" ></i></span>
                                            <div slot="content" class="small-bubble-container">
                                                <div class="input-container">
                                                    <i-input v-model="validateYear"
                                                             @on-keyup="allowInt()"
                                                             class="w120 text-narrow">
                                                        <span slot="append">个</span>
                                                    </i-input>
                                                    <span>自然年</span>
                                                </div>
                                                <div class="grade-btn-container">
                                                    <Button  size="small" type="default" class="text-narrow"
                                                             @click="hideGradeBubble">取消
                                                    </Button>
                                                    <Button size="small" @click="changeValidity"
                                                              type="primary">确定</Button>
                                                </div>
                                            </div>
                                        </Poptip>
                                    </div>
                                </div>
                            </i-col>
                        </Row>
                        <div v-show="grade.modifyTime"
                             class="update-time big-block-narrow">由 {{ grade.modifier }} 在 {{ dateParser(grade.modifyTime) }} 更新</div>
                    </div>
                </i-col>


                <i-col  span="12" class="growth-rule ">
                    <div class="func-card">
                        <div class="card-title">
                            <span>成长值规则</span>
<!--                            <Tooltip placement="top"-->
<!--                                     content="该模式下用户可以存在负成长值"-->
<!--                            >-->
<!--                                <span class="growth-mode">{{ displayGrowthText }}</span>-->
<!--                            </Tooltip>-->

                        </div>

                        <div class="setting-icon">
                            <Tooltip placement="top"
                                     content="成长值设置"
                            >
                            <span @click="jumpGrowthSetting" class="setting-growth">
                                <i  class="icon iconfont iconSetting"  title="设置" ></i>
                            </span>
                            </Tooltip>
                        </div>
                        <Row  class=" big-block-narrow">
                            <i-col span="10" class="func-row">
                                <div class="property-name">成长值有效期</div>
                                <div class="property-value">
                                    <div class="normal-status-text">{{ growthSetting.growthExpirationYear }} 个自然年</div>
                                </div>
                            </i-col>

                            <i-col span="10" class="func-row">
                                <div class="property-name">成长值获取规则</div>
                                <div class="property-value">
                                    <div v-if="growthSetting.open" class="normal-status-text">每消费{{ growthSetting.consumeMoney }}元，获得 1 成长值</div>
                                    <div v-else class="unset-status-text">[未设置]</div>
                                </div>
                            </i-col>
                        </Row>
                        <div v-if="growthSetting.updateTime"
                             class="update-time big-block-narrow">由 {{ growthSetting.modifier }} 在 {{ dateParser(growthSetting.updateTime) }} 更新</div>
                    </div>

                </i-col>
            </Row>

            <div class="table-container middle-block-narrow">

                <div v-if="showLevelModeInfo" class="level-info">
                    <span> <i  class="icon iconfont iconInfo"  title="信息" ></i></span>
                    <span >您已启用等级模式，新会员等级立即生效，老会员等级将于第二日凌晨刷新等级。</span>
                </div>
                <div v-if="!activeLevelMode" class="level-info">
                    <span> <i  class="icon iconfont iconInfo"  title="信息" ></i></span>
                    <span >您已停用等级模式，所有与等级相关的规则均失效。</span>
                </div>


                <Table :columns="columns" class="level-table middle-block-narrow"
                       :loading="tableLoading"
                       :data="levelListData" ></Table>
            </div>

            <Modal
                    v-model="disablePop"
                    footer-hide
            >
                <div class="modal-content">
                    <div class="action-title">
                        <span class="warning-icon"><i  class="icon iconfont iconWarning"  title="信息" ></i></span>
                        <span>等级停用提示</span>
                    </div>
                    <div class="action-content">
                        <div>停用会员等级模式后，将产生如下影响：</div>
                        <div>① 客户仍可获得成长值，但<span class="color-text">不获得等级</span>；</div>
                        <div>② 所有与等级相关的<span class="color-text">权益均失效</span>；</div>
                        <div>③ 若等级规则发生变更，启用后新会员立即生效、<span class="color-text">老会员延迟生效</span>（将在第二天凌晨刷新等级)；</div>
                        <div>④ 若等级规则发生变更，启用后与等级相关的<span class="color-text">权益规则需手动更新</span>匹配最新等级体系。</div>
                        <div>建议在营业结束后进行停用等级模式操作，避免对业务造成影响。是否确认继续停用等级模式？</div>
                    </div>

                    <div class="btn-container huge-block-narrow">
                        <Button @click="closeDisablePop" class="text-narrow">取消</Button>
                        <Button @click="toggleLevelMode('close')" type="primary" >确定</Button>
                    </div>
                </div>
            </Modal>

            <Modal
                    v-model="enableLevelPop"
                    footer-hide
            >
                <div class="modal-content">
                    <div class="action-title">
                        <span class="warning-icon"><i  class="icon iconfont iconWarning"  title="信息" ></i></span>
                        <span>等级启用提示</span>
                    </div>
                    <div class="action-content">
                        <div>启用会员等级模式后，将产生如下影响：</div>
                        <div>① 若等级规则发生变更，启用后与等级相关的权益规则需手动更新匹配最新等级体系；</div>
                        <div>② 若等级规则发生变更，启用后新会员立即生效、<span class="color-text">老会员延迟生效</span>
                            <span>（将在第二天凌晨刷新等级)，延迟期间将</span>
                            <span class="color-text">影响老会员</span>的等级权益。
                        </div>
                        <div>建议在营业结束后进行启用等级模式操作，避免对业务造成影响。是否确认继续启用等级模式？</div>
                    </div>

                    <div class="btn-container huge-block-narrow">
                        <Button @click="closeEnablePop" class="text-narrow">取消</Button>
                        <Button @click="toggleLevelMode('open')" type="primary" >确定</Button>
                    </div>
                </div>
            </Modal>
        </div>
    </div>
</template>

<script>
    import { log, dateParser, _isToday, _isYesterday } from "@/utils/tools";
    import { settingLevelCol } from "./settingLevelCol";

    export default {
        name: "SettingLevel",
        components: { },
        props: {
            list: Array,
            filterName: String,
        },
        data() {
            return {
                // 页面类型：等级编辑 editLevel, 成长值设置 settingGrowth
                pageType: 'settingLevel',
                selectedTagIndex: [],
                columns: settingLevelCol.call(this),
                levelListData: [],

                orgId: '',
                levelIdList: [],
                actionLevelInfo: '',
                tableLoading: false,

                activeLevelMode: false,
                hasConfigLevel: false,

                grade: {
                    id:12,
                    status:1,
                    gradeExpirationYear: '-',
                    modifyTime:"",
                    modifier:""
                },
                gradeBubble: false,
                validateYear: '-',

                // 停用启用灯膜模式的弹窗
                disablePop: false,
                enableLevelPop: false,

                growthSetting: {
                    id: '',
                    consumeMoney: '',
                    createTime: '',
                    description: '',

                    growthExpirationYear: '-',
                    negative: 1,
                    open: false,
                    shareData: "",

                    updateTime: '',
                    modifier:""
                }
            }
        },
        computed: {
            displayGrowthText() {
                let negative = this.growthSetting.negative
                let text = '非负分模式'
                if (negative) {
                    text = '负分模式'
                }
                return text
            },
            showLevelModeInfo() {
                let { modifyTime } = this.grade
                let hour = new Date().getHours()
                let yesterdayUpdate = _isYesterday(modifyTime)
                // 昨天更新，今天的 5 点前显示
                let beforeFive = yesterdayUpdate && (hour <= 5)
                // 今天更新
                let todayUpdate = _isToday(modifyTime)

                let open = this.grade.status === 2
                let show = (todayUpdate || beforeFive ) && open
                return show
            },

        },
        created() {
            this.initData()
            this.getPageInfo()
            this.getGrowth()
        },
        methods: {
            dateParser(time) {
                return dateParser(time)
            },
            initData() {
                let orgId = localStorage.orgId || 'AABBCCDDEEFFGGHH66'
                this.orgId = orgId
            },
            getPageInfo() {
                let server = '/grade-server'
                let url = server + '/api/grade/info'
                let data = { orgId: this.orgId}
                this.tableLoading = true

                this.$ajaxGet(url, data)
                    .then(res => {
                        this.tableLoading = false
                        if (res && res.code === 10000) {
                            let resData = res.data
                            this.renderPage(resData)
                        }
                    })
                    .catch(err => {
                        log(err);
                    });
            },
            getGrowth() {
                let server = '/points-server'
                let url = server + '/api/growth/info'
                let data = { orgId: this.orgId}

                this.$ajaxGet(url, data)
                    .then(res => {
                        if (res && res.code === 10000) {
                            let resData = res.data
                            Object.assign(this.growthSetting, resData)
                        }
                    })
                    .catch(err => {
                        log(err);
                    });
            },
            renderPage(data) {
                let { gradeLevelRelationList, grade } = data
                let tempList = []
                if (gradeLevelRelationList.length > 0) {
                    this.hasConfigLevel = true
                    gradeLevelRelationList.forEach((item) => {
                        tempList.push(item.id)
                    })
                }
                this.levelIdList = tempList
                localStorage.setItem('levelIdList', JSON.stringify(tempList))
                this.activeLevelMode = grade.status === 2

                if ((gradeLevelRelationList.length < 10) && !this.activeLevelMode) {
                    let lastRow = {
                        id: '-',
                        levelId: `Lv${gradeLevelRelationList.length + 1}`,
                        cardBackground: '',
                        levelName: '-',
                        orgId: '',
                        protectionGradeValue: '-',
                        upGradeValue: '-',
                    }
                    gradeLevelRelationList.push(lastRow)
                }

                this.levelListData = gradeLevelRelationList
                localStorage.setItem('enableLevel', JSON.stringify(this.activeLevelMode))
                Object.assign(this.grade, grade)
                this.validateYear = grade.gradeExpirationYear

            },
            isLastLevel(id) {
                let lastInList = this.levelIdList[this.levelIdList.length - 1]
                return id === lastInList
            },
            createLevelName(levelId) {
                return `${levelId.slice(0, 2).toUpperCase()} ${levelId.slice(2)}`
            },
            tableAction(type, item) {
                this.actionLevelInfo = item
                let handlers = {
                    edit: this.configLevel,
                    delete: this.deleteLevel,
                }
                let fn = handlers[type]
                if (fn) {
                    fn()
                }
            },
            configLevel() {
                localStorage.setItem('levelInfo', JSON.stringify(this.actionLevelInfo))
                this.$router.push(`/member/level/editLevel`)
            },
            deleteLevel() {
                let server = '/grade-server'
                let url = server + '/api/level/delete'
                let data = { id: this.actionLevelInfo.id }

                this.$ajaxGet(url, data)
                    .then(res => {
                        if (res && res.code === 10000) {
                            this.$Message.success(res.message)
                            this.getPageInfo()
                        }
                    })
                    .catch(err => {
                        log(err);
                    });
            },
            hideGradeBubble() {
                this.value = 0
                this.gradeBubble = false
            },
            toggleLevelMode(type) {
                let status = {
                    open: '2',
                    close: '1'
                }
                this.grade.status= status[type]
                this.closeDisablePop()
                this.closeEnablePop()
                this.modifyLevelMode()
            },
            validateForm() {
                if (this.validateYear === undefined) {
                    this.$Message.error('操作失败，未填写等级有效期！')
                    return true
                }

                return false
            },
            modifyLevelMode() {
                let server = '/grade-server'
                let url = server + '/api/grade/edit'
                let notPass = this.validateForm()
                if (notPass) {
                    return
                }
                let data = {
                    orgId: this.orgId,
                    status: this.grade.status,
                    gradeExpirationYear: this.validateYear,
                }

                this.$ajaxPost(url, data)
                    .then(() => {
                        this.getPageInfo()
                    })
                    .catch(err => {
                        log(err);
                    });
            },
            changeValidity() {
                this.gradeBubble = false
                this.modifyLevelMode()
            },

            allowInt() {
                let currentValue = this.validateYear
                let temp
                if (currentValue.length === 1) {
                    temp = currentValue.replace(/[^1-9]/g, '')
                } else {
                    temp = currentValue.replace(/\D/g, '')
                }

                let big = Number(temp) > 99
                if (big) {
                    temp = String(99)
                }

                this.validateYear = temp
            },

            jumpGrowthSetting() {
                this.$router.push(`/member/level/editGrowth`)
            },
            closeDisablePop() {
                this.disablePop = false
            },
            openDisablePop() {
                if (!this.hasConfigLevel) {
                    this.$Message.error('配置等级后，才能调整模式状态！')
                    return
                }
                this.disablePop = true
            },
            closeEnablePop() {
                this.enableLevelPop = false
            },
            openEnablePop() {
                if (!this.hasConfigLevel) {
                    this.$Message.error('配置等级后，才能调整模式状态！')
                    return
                }
                this.enableLevelPop = true
            },
        }
    }
</script>

<style lang="scss" scoped>
    .setting-level {
        padding-right: 12px;
    }
    .member-level {
        padding-right: 12px;
    }

    .growth-rule {
        padding-left: 12px;
    }

    .func-card {
        position: relative;
        background:rgba(255,255,255,1);
        box-shadow:0px 2px 12px 0px rgba(0,0,0,0.04);
        border-radius: 20px;

        padding: 24px;
        min-height: 200px;
        box-sizing: border-box;
    }

    .setting-icon {
        position: absolute;
        display: inline-block;
        right: 24px;
        top: 24px;
    }

    .card-title {
        & > span {
            font-size:16px;
            font-weight:500;
            color:rgba(23,23,37,1);
            line-height:26px;
        }

    }

    .growth-mode {
        margin-left: 8px;
        background:rgba(246,246,247,1);
        border-radius:11px;
        border:1px solid rgba(213,213,220,1);

        padding: 0 12px;
        font-size:12px;
        font-weight:300;
        color:rgba(105,105,116,1);
        line-height:22px;
    }

    .setting-growth {

        .iconfont {
            cursor: pointer;
            color: #FF8C08;
        }
    }
    .func-row {
        margin-right: 24px;
    }

    .big-block-narrow {
        margin-top: 24px;
    }

    .middle-block-narrow {
        margin-top: 20px;
    }

    .huge-block-narrow {
        margin-top: 40px;
    }

    .property-name {
        font-size:12px;
        font-weight:300;
        color:rgba(146,146,157,1);
        line-height:22px;
    }

    .property-value {
        margin-top: 4px;
        color:rgba(23,23,37,1);

    }

    .normal-status-text {
        font-size:14px;
        line-height:24px;
        &  span {
            display: inline-block;
            vertical-align: middle;
            height: 24px;
        }

        .iconEdit {
            display: none;
        }
        .iconEdit:hover {
            cursor: pointer;
            color: #FF8C08;
        }
    }

    .func-card:hover .iconEdit {
        display: inline-block;
    }

    .status-icon {
        display: inline-block;
        position: relative;
        bottom: 2px;
        width:6px;
        height:6px;
        border-radius: 6px;
        background:rgba(80,181,255,1);
    }

    .status-icon-disable {
        display: inline-block;
        position: relative;
        bottom: 2px;
        width:6px;
        height:6px;
        border-radius: 6px;
        background:rgba(181,181,190,1);
    }

    .status-text {
        /*border-right: 1px solid rgba(226,226,234,1);*/
        display: inline-block;
        position: relative;
        font-size:14px;

        line-height:24px;
        padding: 0 8px;

        &:after {
            content: '';
            position: absolute;
            right: 0;
            top: 5px;
            bottom: 5px;
            width: 1px;
            background-color: rgba(226,226,234,1);
        }
    }

    .status-close {
        display: inline-block;
        padding: 0 8px;
        &:active {
            color: #a65308;
        }
    }

    .unset-status-text {
        font-size:14px;
        color:rgba(146,146,157,1);
        line-height:24px;
    }

    .action-text {
        cursor: pointer;
        color: #FF8C08;
    }

    .color-text {
        color:rgba(255,140,8,1);
    }

    .update-time {
        font-size:10px;
        font-weight:300;
        color:rgba(146,146,157,1);
        line-height:20px;
    }

    .table-container {
        padding: 24px;
        background:rgba(255,255,255,1);
        box-shadow:0px 2px 12px 0px rgba(0,0,0,0.04);
        border-radius:16px;
    }

    .level-info {
        background:rgba(255,246,236,1);
        border-radius:4px;
        border:1px solid rgba(255,140,8,1);
        padding: 12px 16px;

        font-size:14px;
        font-weight:300;
        color:rgba(23,23,37,1);
        line-height:24px;

        .iconInfo {
            color: rgba(255,140,8,1);
        }
    }

    .text-narrow {
        margin-right: 16px;
    }

    .grade-btn-container {
        display: flex;
        justify-content: flex-end;
        margin-top: 16px;
    }

    .small-bubble-container {
        width: 200px;
        padding: 16px;
    }

    .input-container {
        display: flex;
        justify-content: center;

        & > span {
            line-height: 32px;
        }
    }

    .modal-content {
        padding: 16px;
        position: relative;
    }

    .action-title {
        font-size:18px;
        font-family:SFUIDisplay-Medium,SFUIDisplay;
        font-weight: bold;
        color:rgba(23,23,37,1);
        line-height:28px;
        position: relative;

        .iconWarning {
            color: #ffc542;
        }
    }

    .warning-icon {
        position: absolute;
        left: -24px;
    }

    .action-content {
        margin-top: 8px;
        font-size:14px;
        color:rgba(68,68,79,1);
        line-height:24px;
    }

    .btn-container {
        display: flex;
        justify-content: flex-end;
    }
</style>
