    <template>
    <div class="step-four">
        <div class="row m-0 p-0 mb-5">
            <div class="col-md-8 m-auto">
                <div class="text-center mb-3">
                    <div class="row">
                        <div class="col-3"></div>
                        <div class="col-5 ml-5 mb-2 text-center">
                            <button class="btn-custom mr-3" @click="canvas.discardActiveObject().renderAll();isOpen=true">미 리 보 기</button>
                            <!-- <button v-if="isWork" class="btn-custom mr-2" @click="saveWork">저 장 하 기</button> -->
                        </div>
                        <div class="col-3">
                            <div class="h-100 alert-icon d-flex justify-content-center align-items-center">
                                <span><i class="mdi mdi-alert-circle-outline"></i></span> &nbsp;
                                <small>주의사항</small>
                                <div class="alert-msg">
                                    <div class="text-center text-danger">
                                        <strong>실제 효과들은 사진 뒤에 표시됩니다. <br> 미리보기 버튼을 통해 확인할 수 있습니다.</strong>
                                    </div>
                                </div>
                            </div>
                        </div>
                    </div>
                </div>
                <div class="mb-3 d-flex justify-content-center">
                    <div ref="pic" :class="`outter-frame-${parseInt(frame.split('x')[0])}-${parseInt(frame.split('x')[1])}`" style="position: absolute; padding: 20px; padding-right: 0px;">
                        <div :class="`row p-0 m-0`" v-for="(row, rowIdx) of rowCnt" :key="rowIdx">
                            <div :class="`pl-0 pr-0 inner-frame inner-frame-${columns}-${rows}`" v-for="(col, colIdx) of colCnt" :key="colIdx">
                                <div :class="`inner-frame-${columns}-${rows}`">
                                    <img :src="images[rowIdx*colCnt.length + col]" :id="`canvas-${rowIdx*colCnt.length + col}`" draggable="false">
                                </div>
                            </div>
                        </div>
                    </div>
                    <div ref="deco" @drop="drop" @dragover.prevent>
                        <canvas v-if="frame" :class="`outter-frame outter-frame-${parseInt(frame.split('x')[0])}-${parseInt(frame.split('x')[1])}`" ref="canvas"></canvas>
                    </div>
                </div>
                <div class="mb-3 text-center" v-if="isWork"><strong>Delete 키로 스티커를 지울 수 있습니다.</strong></div>
            </div>
            <div class="col-md-4">
                <span :class="{ 'target': isMode=='bg' }" class="p-2" @click="isWork=false;setWorkMode();isMode='bg'">배경색</span> | 
                <span :class="{ 'target': isMode=='sticker' }" class="p-2" @click="isWork=true;setWorkMode();isMode='sticker'">스티커</span>
                <hr>
                <div :style="`height: 600px;overflow-y: auto;`">
                    <div v-if="isMode=='bg'">
                        <div v-for="(value, theme) in bg" :key="theme">
                            <div class="mb-3"><strong>{{theme}} 테마</strong></div>
                            <div class="row m-0 p-0 mb-3">
                                <div :class="{ 'target': targetColor == color }" class="col-md-3 m-0 p-0 mb-3 pt-1 pb-1" v-for="(color, idx) of value" :key="idx">
                                    <div class="bg m-auto" :style="{'background-color': color}" @click="selectBg(color)"></div>
                                </div>
                            </div>
                        </div>
                        <div v-for="(value, theme) in pattern" :key="theme">
                            <div class="mb-3"><strong>패턴</strong></div>
                            <div class="row m-0 p-0 mb-5">
                                <div :id="`pattern_${item}`" :class="{ 'target': targetPattern == `pattern_${item}` }" class="col-md-3 m-0 p-0 mb-3 pt-1 pb-1 text-center" v-for="(item, idx) of value" :key="idx">
                                    <img :ref="`pattern_${item}`" class="pattern" :src="require(`@/assets/pattern/pattern_${item}.png`)" @click="selectPattern(`pattern_${item}`)" draggable="false">
                                </div>
                            </div>
                        </div>
                    </div>
                    <div v-else-if="isMode=='sticker'">
                        <div class="mb-5">
                            <button :class="{'btn-primary': isText, 'btn-outline-primary': !isText}" class="btn mr-3" @click="isText=!isText">텍스트</button>
                            <input type="number" v-model="fontSize">
                        </div>
                        <div>
                            <div v-for="(value, theme) in sticker" :key="theme">
                                <div class="mb-3"><strong>{{theme}} 테마</strong></div>
                                <div class="row m-0 p-0 mb-5">
                                    <div :id="`${theme}_${item}`" :class="{ 'target': targetSticker == `${theme}_${item}` }" class="col-3 mb-3 text-center" v-for="(item, idx) of value" :key="idx" @click="isSticker=!isSticker;selectSticker(`${theme}_${item}`)">
                                        <img :ref="`${theme}_${item}`" class="sticker" :src="require(`@/assets/stickers/${theme}_${item}.png`)">
                                    </div>
                                </div>
                            </div>
                        </div>
                    </div>
                </div>    
            </div>
        </div>
        <preview-modal v-if="isOpen" @on-close="isOpen=false" :columns="parseInt(frame.split('x')[0])" :rows="parseInt(frame.split('x')[1])"></preview-modal>
    </div>
</template>

<script>
import { fabric } from 'fabric'
import previewModal from './vues/previewModal.vue'
import previewFrame from './frames/previewFrame.vue'

export default {
    name: 'stepFour',
    components: {
        previewModal,
        previewFrame,
    },
    data() {
        return {
            rows: 2,
            columns: 1,
            rowCnt: [],
            colCnt: [],
            images: {},
            targetSticker: null,
            targetColor: '#00ff0000',
            targetPattern: null,
            frame: null,
            isMode: 'bg',
            isWork: false,
            isOpen: false,
            isText: false,
            isSticker: false,
            canvasHeight: null,
            canvasWidth: null,
            canvas: null,
            fontSize: 16,
            contentHeight: 500,
            sticker: {
                'cute_handdrawn': [],
                'cute_natural_doodle': [],
                'flower_leaf': [],
            },
            pattern: {
                'basic': [],
            },
            bg: {
                'simple': ['#F2F2F3', '#A6A6A6', '#595959', '#262626', '#0D0D0D'],
                'modern': ['#131B26', '#D9B95B', '#D9C484', '#F2ECE4', '#D97D5B'],
                'warm': ['#D9C077', '#F29F05', '#D97904', '#BF4904', '#F2F2F2'],
                'astro': ['#F25E7A', '#4A2B8C', '#5155A6', '#05F2DB', '#F2E963'],
                'cartoon': ['#636AF2', '#41A0F2', '#A2DCF2', '#04D98B', '#F2E205'],
                'ancient': ['#1D5948', '#F2BF5E', '#A6864B', '#F2D091', '#732509'],
            }
        }
    },
    created() {
        this.frame = this.$store.getters.getFrame;
    },
    mounted() {
        this.init();
        window.addEventListener('keydown', this.setKeydownEvent);
    },
    beforeDestroy() {
        this.saveWork();
    },
    destroyed() {
        window.removeEventListener('keydown', this.setKeydownEvent);
    },
    methods: {
        async init() {
            let table = this.$store.getters.getTable;

            this.images = this.$store.getters.getTargets;
            this.rows = table.rows;
            this.columns = table.columns;
            this.rowCnt = Array.from({length: table.rows}, (v, i) => i + 1);
            this.colCnt = Array.from({length: table.columns}, (v, i) => i + 1);
            this.sticker.cute_handdrawn = Array.from({length: 6}, (v, i) => i + 1);
            this.sticker.cute_natural_doodle = Array.from({length: 12}, (v, i) => i + 1);
            this.sticker.flower_leaf = Array.from({length: 38}, (v, i) => i + 1);
            this.pattern.basic = Array.from({length: 9}, (v, i) => i + 1);

            this.canvasHeight = this.$refs.canvas.clientHeight;
            this.canvasWidth = this.$refs.canvas.clientWidth;
            this.canvas = new fabric.Canvas(this.$refs.canvas);
            this.canvas.setHeight(this.canvasHeight);
            this.canvas.setWidth(this.canvasWidth);

            if (this.$store.getters.getCanvas) {
                await this.loadCanvasToJSON();
                this.setCanvasOption();
                this.targetColor = this.canvas.backgroundColor;
                this.canvas.renderAll();
            }

            this.setMouseEvent();
            this.setWorkMode();
        },
        loadCanvasToJSON() {
            return new Promise((resolve, reject) => {
                this.canvas.loadFromJSON(this.$store.getters.getCanvas, this.canvas.renderAll.bind(this.canvas));
                resolve();
            })
        },
        setMouseEvent() {
            this.canvas.on('mouse:down', (e) => {
                if (!this.isWork) return;

                if (this.isText) {
                    this.createObj('text', e.pointer.x, e.pointer.y);
                } else if (this.isSticker) {
                    this.createObj('sticker', e.pointer.x, e.pointer.y);
                } else if (e.target) {
                    // e.target.opacity = 0.8;
                    this.canvas.renderAll();
                }
            }).on('mouse:up', (e) => {
                if (e.target) {
                    e.target.opacity = 1;
                    this.canvas.renderAll();
                }
            }).on('selection:created', (e) => {
                if (e.selected.length > 1) this.canvas.discardActiveObject().renderAll();
            }).on('selection:cleared', (e) => {
                console.log(e.target, 'cleared')
                this.saveWork();
                this.canvas.discardActiveObject().renderAll();
            })
        },
        setKeydownEvent(e) {
            const allowKeys = [' ', 'DELETE']
            let pressKey = e.key.toUpperCase();

            if (!allowKeys.includes(pressKey)) return false
                
            if (pressKey == 'DELETE') {
                let targetObj = this.canvas.getActiveObject();

                this.canvas.remove(targetObj);
                this.canvas.renderAll();

                this.saveWork();
            }
        },
        createObj(type, left, top) {
            if (type == 'text') {
                let textBox = new fabric.Textbox('내용을 입력하세요.', {
                    left: left,
                    top: top,
                    width: 150,
                    fontSize: this.fontSize,
                    textAlign: 'left',
                    id: `textBox_${(new Date).getDate()}`
                });

                this.canvas.add(textBox);
                this.canvas.renderAll();
                
                this.setTextEvent(textBox);

                this.isText = false;
            } else if (type == 'sticker') {
                const sticker = this.$refs[this.targetSticker][0]
                let image = new fabric.Image(sticker, {
                    left: left,
                    top: top,
                    borderColor: 'red',
                    cornerColor: 'green',
                    cornerSize: 10,
                    transparentCorners: false,
                });
                
                image.left = left - (image.width / 2);
                image.top = top - (image.height / 2);

                this.canvas.add(image);
                this.canvas.renderAll();

                this.isSticker = false;
                this.targetSticker = null;
            }

            this.saveWork();
        },
        createDragObj(type, target, left, top) {
            if (type == 'sticker') {
                fabric.Image.fromURL(target,  (img) => {
                    img.left = left - (img.width / 2);
                    img.top = top - (img.height / 2);
                    img.borderColor = 'red',
                    img.cornerColor = 'green',
                    img.cornerSize = 10,
                    img.transparentCorners = false,

                    this.canvas.add(img);
                    this.canvas.renderAll();
                    
                    this.saveWork();
                });

                this.isSticker = false;
                this.targetSticker = null;
            }
        },
        saveWork() {
            this.canvas.discardActiveObject().renderAll();

            this.setWorkMode();
            this.$store.commit('setCanvas', JSON.stringify(this.canvas.toObject(['id', 'borderColor', 'cornerColor', 'cornerSize', 'transparentCorners'])));
            this.$store.commit('setFrameImg', this.canvas.toDataURL({ format: 'image/png' }));
        },
        setTextEvent(textBox) {
            textBox.setControlsVisibility({
                mt: false, mb: false, 
                bl: false, br: false, 
                tl: false, tr: false,
            });

            textBox.on('selected', (e) => {
                this.canvas.renderAll();
            });

            textBox.on("editing:entered", (e) =>{
                let obj = this.canvas.getActiveObject();

                if (obj.text == "내용을 입력하세요."){
                    obj.selectAll();
                    obj.removeChars();
                }
            });

            textBox.on(("changed"), () => {

                let actualWidth = textBox.scaleX * textBox.width;
                let largest = this.canvas.getActiveObject().__lineWidths.filter(item => item)[0] ? this.canvas.getActiveObject().__lineWidths.filter(item => item)[0] : 1;
                let tryWidth = (largest + 15) * textBox.scaleX;

                this.canvas.getActiveObject().set("width", tryWidth);

                if ((textBox.left + actualWidth) >= this.canvas.width - 10) {
                    textBox.set("width", this.canvas.width - left - 10)
                }

                this.canvas.renderAll();
            });

            textBox.on(("modified"), () => {
                this.canvas.renderAll();
            });
        },
        setCanvasOption() {
            this.canvas.getObjects().map(item => {
                if (item.id.includes('textBox')) {
                    this.setTextEvent(item);
                }
            })
        },
        selectSticker(id) {
            if (this.targetSticker == id) this.targetSticker = null;
            else this.targetSticker = id;
        },
        setWorkMode() {
            this.canvas.discardActiveObject().renderAll();

            if (this.isWork) {
                this.$refs.pic.style['z-index'] = 1;
                this.$refs.deco.style['z-index'] = 2;
                this.$refs.deco.style['opacity'] = 0.8;
            } else {
                this.$refs.pic.style['z-index'] = 2;
                this.$refs.deco.style['z-index'] = 1;
                this.$refs.deco.style['opacity'] = 1;
            }
        },
        selectBg(color) {
            if (this.targetColor == color) {
                this.targetColor = '#FFF';
                this.canvas.backgroundColor = '#FFF';
            } else {
                this.targetColor = color;
                this.targetPattern = null;
                this.canvas.backgroundColor = color;
            }
            
            this.canvas.renderAll();

            this.saveWork();
        },
        selectPattern(pattern) {
            if (this.targetPattern == pattern) {
                this.canvas.backgroundColor = '#FFF';
                this.targetColor = '#FFF';
                this.targetPattern = null;
                this.saveWork();
            } else {
                this.targetColor = '#FFF'
                this.targetPattern = pattern;

                this.canvas.setBackgroundColor({ source: this.$refs[this.targetPattern][0].src, repeat: 'repeat' }, () => {
                    this.canvas.renderAll();
                    this.saveWork();
                });
            }

            this.canvas.renderAll();
        },

        drop(e) {
            const dragImg = e.dataTransfer.getData('text/plain');
            this.createDragObj('sticker', dragImg, e.offsetX, e.offsetY);
        },
    },
    computed: {
        getContentHeight() {
            return window.screen.availHeight;
        }
    },
}
</script>

<style lang="scss" scoped>
.frame {
    height: 630px;
}

img {
    object-position: center center;
    object-fit: cover;
    height: 100%;
    width: 100%;
}

.bg {
    height: 40px;
    width: 40px;
}

.sticker, .pattern {
    object-fit: cover;
    height: 40px;
    width: 40px;
}

.outter-frame {
    padding-right: 0px;
    padding-bottom: 0px;
    box-shadow: 0.5px 0.5px 1.5px black;

    &-1-1 {
        height: 350px;
        width: 400px;
    }
    &-1-2 {
        height: 520px;
        width: 320px;
    }
    &-1-3 {
        height: 525px;
        width: 220px;
    }
    &-1-4 {
        height: 560px;
        width: 180px;
    }
    &-2-1 {
        height: 320px;
        width: 520px;
    }
    &-2-2 {
        height: 520px;
        width: 620px;
    }
    &-2-3 {
        height: 525px;
        width: 420px;
    }
    &-3-1 {
        height: 220px;
        width: 525px;
    }
    &-3-2 {
        height: 420px;
        width: 525px;
    }
    &-4-1 {
        height: 180px;
        width: 560px;
    }
}

.inner-frame {
    position: relative;
    margin-bottom: 20px;
    margin-right: 20px;
    background: #FFF;
    box-shadow: 0.5px 0.5px 1.5px black;

    &-1-1 {
        height: 270px;
        width: 360px;
    }
    &-1-2 {
        height: 210px;
        width: 280px;
    }
    &-1-3 {
        height: 135px;
        width: 180px;
    }
    &-1-4 {
        height: 105px;
        width: 140px;
    }
    &-2-1 {
        height: 280px;
        width: 210px;
    }
    &-2-2 {
        height: 210px;
        width: 280px;
    }
    &-2-3 {
        height: 135px;
        width: 180px;
    }
    &-3-1 {
        height: 180px;
        width: 135px;
    }
    &-3-2 {
        height: 180px;
        width: 135px;
    }
    &-4-1 {
        height: 140px;
        width: 105px;
    }
}

.target {
    background-color: grey;
}

.alert-msg {
    position: absolute;
    opacity: 0;
}

.alert-icon:hover .alert-msg {
    padding: 20px;
    width: 400px;
    top: -10;
    left: -10;
    z-index: 3;
    background-color: #FFF;
    border: 1px black solid;

    animation: moveFromRight 0.7s ease both;

    @keyframes moveFromRight {
        to {
            opacity: 1;
            transform: translateY(10%);
        }
    }   
}
</style>