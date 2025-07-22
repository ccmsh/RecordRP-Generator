<template>
    <div class="navbar bg-base-100 shadow-sm">
        <div class="flex-1">
            <a class="btn btn-ghost text-xl">レコードを置き換える</a>
        </div>
        <div class="flex-none">
            <div class="dropdown dropdown-end">
                <a role="button" class="btn btn-ghost btn-circle" href="https://github.com/ccmsh/RecordRP-Generator">
                    <div class="w-10 rounded-full">
                        <img id="github" alt="Github Logo" src="/github-mark/github-mark.png" />
                    </div>
                </a>
            </div>
        </div>
    </div>
    <div style="margin: 2%;">
        <p class="text-lg font-semibold ml-2">Minecraftバージョンを選択</p>
        <select class="select select-bordered w-full max-w-xs ml-2" v-model="packFormat">
            <option value="34">1.21</option>
            <option value="22">1.20.5 / 1.20.6</option>
            <option value="15">1.20.1</option>
            <option value="13">1.19.4</option>
            <option value="12">1.19.3</option>
            <option value="10">1.19.x</option>
            <option value="9">1.18.x</option>
            <option value="8">1.17.x</option>
            <option value="7">1.16.x</option>
            <option value="6">1.15.x</option>
            <option value="5">1.14.x</option>
            <option value="4">1.13.x</option>
            <option value="3">1.11.x / 1.12.x</option>
            <option value="2">1.9.x / 1.10.x</option>
            <option value="1">1.8.x</option>
        </select>

        <p class="text-lg font-semibold ml-2 mt-4">リソースパックのアイコンを選択</p>
        <input type="file" class="file-input file-input-bordered file-input-sm w-full max-w-xs ml-2"
            @change="handleIconUpload" accept="image/png, image/jpeg" />

        <p class="text-lg font-semibold ml-2">置き換えるレコードを選択</p>
        <div>
            <label v-for="record in records" :key="record" class="btn btn-square p-1 ml-2"
                :class="{ 'btn-active btn-info': selected.includes(record) }" style="cursor:pointer;">
                <input type="checkbox" class="hidden" :id="record" :value="record" v-model="selected">
                <img :src="`/records/${record}.png`" :alt="record">
            </label>
        </div>
        <div class="ml-2 mt-2">
            <p class="text-lg font-semibold">置き換える音楽ファイルを設定</p>
            <div v-for="record in selected" :key="record" class="flex items-center mt-2">
                <span class="w-24">{{ record }}</span>
                <input type="file" class="file-input file-input-bordered file-input-sm w-full max-w-xs mr-2"
                    @change="handleFileUpload(record, $event)" accept=".ogg,.mp3,.wav" />
                <span class="text-sm">
                    <template v-if="converting[record]">
                        変換中... {{ convertProgress[record] || 0 }}%
                        <progress class="progress progress-info w-24" :value="convertProgress[record] || 0" max="100"></progress>
                    </template>
                    <template v-else-if="convertError[record]">
                        <span class="text-error">{{ convertError[record] }}</span>
                    </template>
                    <template v-else>
                        {{ uploadedFiles[record] ? uploadedFiles[record].name : 'ファイルが選択されていません' }}
                    </template>
                </span>
            </div>
        </div>
        <div class="ml-2 mt-4">
            <button class="btn btn-primary" @click="generatePack" :disabled="!isReadyToGenerate">リソースパックを生成</button>
        </div>
    </div>
</template>
<script>
export default {
    data() {
        return {
            selected: [],
            uploadedFiles: {},
            packFormat: 34, // Default to latest version
            croppedIcon: null,
            converting: {},
            convertProgress: {}, // 追加: 進捗率管理
            convertError: {},    // 追加: エラー管理
            versionRecordsMap: {
                1: ['11', '13', 'blocks', 'cat', 'chirp', 'far', 'mall', 'mellohi', 'stal', 'strad', 'wait', 'ward'],
                2: ['11', '13', 'blocks', 'cat', 'chirp', 'far', 'mall', 'mellohi', 'stal', 'strad', 'wait', 'ward'],
                3: ['11', '13', 'blocks', 'cat', 'chirp', 'far', 'mall', 'mellohi', 'stal', 'strad', 'wait', 'ward'],
                4: ['11', '13', 'blocks', 'cat', 'chirp', 'far', 'mall', 'mellohi', 'stal', 'strad', 'wait', 'ward'],
                5: ['11', '13', 'blocks', 'cat', 'chirp', 'far', 'mall', 'mellohi', 'stal', 'strad', 'wait', 'ward'],
                6: ['11', '13', 'blocks', 'cat', 'chirp', 'far', 'mall', 'mellohi', 'stal', 'strad', 'wait', 'ward'],
                7: ['11', '13', 'blocks', 'cat', 'chirp', 'far', 'mall', 'mellohi', 'stal', 'strad', 'wait', 'ward', 'pigstep'],
                8: ['11', '13', 'blocks', 'cat', 'chirp', 'far', 'mall', 'mellohi', 'stal', 'strad', 'wait', 'ward', 'pigstep', 'otherside'],
                9: ['11', '13', 'blocks', 'cat', 'chirp', 'far', 'mall', 'mellohi', 'stal', 'strad', 'wait', 'ward', 'pigstep', 'otherside', '5'],
                10: ['11', '13', 'blocks', 'cat', 'chirp', 'far', 'mall', 'mellohi', 'stal', 'strad', 'wait', 'ward', 'pigstep', 'otherside', '5', 'relic'],
                12: ['11', '13', 'blocks', 'cat', 'chirp', 'far', 'mall', 'mellohi', 'stal', 'strad', 'wait', 'ward', 'pigstep', 'otherside', '5', 'relic'],
                13: ['11', '13', 'blocks', 'cat', 'chirp', 'far', 'mall', 'mellohi', 'stal', 'strad', 'wait', 'ward', 'pigstep', 'otherside', '5', 'relic'],
                15: ['11', '13', 'blocks', 'cat', 'chirp', 'far', 'mall', 'mellohi', 'stal', 'strad', 'wait', 'ward', 'pigstep', 'otherside', '5', 'relic', 'creator', 'creator_diskbox'],
                22: ['11', '13', 'blocks', 'cat', 'chirp', 'far', 'mall', 'mellohi', 'stal', 'strad', 'wait', 'ward', 'pigstep', 'otherside', '5', 'relic', 'creator', 'creator_diskbox'],
                34: ['11', '13', 'blocks', 'cat', 'chirp', 'far', 'mall', 'mellohi', 'stal', 'strad', 'wait', 'ward', 'pigstep', 'otherside', '5', 'relic', 'creator', 'creator_diskbox', 'lavachicken']
            }
        }
    },
    computed: {
        records() {
            return this.versionRecordsMap[this.packFormat] || [];
        },
        isReadyToGenerate() {
            if (this.selected.length === 0) {
                return false;
            }
            for (const record of this.selected) {
                if (!this.uploadedFiles[record]) {
                    return false;
                }
            }
            return true;
        }
    },
    watch: {
        packFormat(newVal, oldVal) {
            // Clear selected and uploaded files when packFormat changes
            this.selected = [];
            this.uploadedFiles = {};
        }
    },
    methods: {
        async handleIconUpload(event) {
            const file = event.target.files[0];
            if (!file) return;

            const reader = new FileReader();
            reader.onload = (e) => {
                const img = new Image();
                img.onload = () => {
                    const canvas = document.createElement('canvas');
                    canvas.width = 32;
                    canvas.height = 32;
                    const ctx = canvas.getContext('2d');
                    ctx.drawImage(img, 0, 0, 32, 32);
                    canvas.toBlob((blob) => {
                        this.croppedIcon = blob;
                    });
                };
                img.src = e.target.result;
            };
            reader.readAsDataURL(file);
        },
        async handleFileUpload(record, event) {
            const file = event.target.files[0];
            if (!file) return;

            if (file.name.endsWith('.mp3') || file.name.endsWith('.wav')) {
                this.converting[record] = true;
                this.convertProgress[record] = 0;
                this.convertError[record] = '';
                if (process.client) {
                    try {
                        const { FFmpeg } = await import('@ffmpeg/ffmpeg');
                        const { fetchFile, toBlobURL } = await import('@ffmpeg/util');
                        const baseURL = 'https://unpkg.com/@ffmpeg/core@0.12.6/dist/umd'
                        const coreURL = await toBlobURL(`${baseURL}/ffmpeg-core.js`, 'text/javascript');
                        const wasmURL = await toBlobURL(`${baseURL}/ffmpeg-core.wasm`, 'application/wasm');
                        const workerURL = coreURL;

                        const ffmpeg = new FFmpeg({ coreURL, wasmURL, workerURL });

                        ffmpeg.on('progress', ({ ratio }) => {
                            this.convertProgress[record] = Math.round((ratio || 0) * 100);
                        });

                        await ffmpeg.load();
                        await ffmpeg.writeFile(file.name, await fetchFile(file));
                        await ffmpeg.exec(['-i', file.name, 'output.ogg']);
                        const data = await ffmpeg.readFile('output.ogg');
                        this.uploadedFiles[record] = new File([data.buffer], `${record}.ogg`, { type: 'audio/ogg' });
                    } catch (error) {
                        this.convertError[record] = '変換失敗: ' + (error?.message || error);
                        this.uploadedFiles[record] = null;
                    }
                } else {
                    this.uploadedFiles[record] = null;
                }
                this.converting[record] = false;
                this.convertProgress[record] = 100;
            } else {
                this.uploadedFiles[record] = file;
                this.convertProgress[record] = 100;
                this.converting[record] = false;
            }
        },
        async generatePack() {
            const { default: JSZip } = await import('jszip');
            const zip = new JSZip();

            if (this.croppedIcon) {
                zip.file('pack.png', this.croppedIcon);
            } else {
                const packPng = await fetch('/pack.png').then(res => res.blob());
                zip.file('pack.png', packPng);
            }
            zip.file('pack.mcmeta', JSON.stringify({
                pack: {
                    pack_format: this.packFormat,
                    description: 'Custom Music Discs by RecordRP-Generator'
                }
            }));
            const assets = zip.folder('assets');
            const minecraft = assets.folder('minecraft');
            const sounds = minecraft.folder('sounds');
            const records = sounds.folder('records');

            for (const record in this.uploadedFiles) {
                const file = this.uploadedFiles[record];
                records.file(`${record}.ogg`, file);
            }

            const content = await zip.generateAsync({ type: 'blob' });
            const link = document.createElement('a');
            link.href = URL.createObjectURL(content);
            link.download = 'RecordRP.zip';
            link.click();
        }
    }
}
</script>

<style>
.btn-square {
    width: 6rem;
    height: 6rem;
}

@media (prefers-color-scheme: dark) {
    #github {
        content: url("/github-mark/github-mark-white.svg");
    }
}

@media (prefers-color-scheme: light) {
    #github {
        content: url("/github-mark/github-mark.svg");
    }
}
</style>