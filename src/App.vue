<script setup>
import { ref, computed, watch } from 'vue';
import Numeric from './components/Numeric.vue';

const strings = ['E', 'B', 'G', 'D', 'A', 'E'];
const notes_flat = ['C', 'Db', 'D', 'Eb', 'E', 'F', 'Gb', 'G', 'Ab', 'A', 'Bb', 'B'];
const notes_sharp = ['C', 'C#', 'D', 'D#', 'E', 'F', 'F#', 'G', 'G#', 'A', 'A#', 'B'];

const fret_start = ref(0);
const fret_end = ref(12);
const current_note = ref('C');
const accent_color = ref('#8a4bd8');
const selected_category_key = ref('major');
const selected_type_key = ref('major_scale');
const use_flat = ref(false);
const use_degree = ref(false);

const notes = ref(notes_sharp);

watch(use_flat, () => {
    let note = current_note.value;
    current_note.value = use_flat.value ? sharp_to_flat(note) : flat_to_sharp(note);
    notes.value = use_flat.value ? notes_flat : notes_sharp;
});

const frets = computed(() => {
    const list = [];
    for (let fret = fret_start.value; fret <= fret_end.value; fret++) {
        list.push(fret);
    }
    return list;
});

const current_category_types = computed(() => {
    const types = categories[selected_category_key.value].types;
    selected_type_key.value = Object.keys(types)[0];
    return types;
});

const current_scale_data = computed(() => {
    return current_category_types.value[selected_type_key.value];
});

const scale_note_set = computed(() => {
    const set = new Set();
    const scale = current_scale_data.value;
    if (!scale) {
        return set;
    }

    const root_ndex = notes.value.indexOf(current_note.value);
    if (root_ndex === -1) return set;

    let current_index = root_ndex;
    set.add(notes.value[current_index]);

    for (let interval of scale.intervals) {
        current_index = (current_index + interval) % 12;
        set.add(notes.value[current_index]);
    }

    return set;
});

function has_marker(fret_index) {
    const markers = [3, 5, 7, 9, 12, 15, 17, 19, 21];
    return markers.includes(fret_index);
}

function get_note(string_index, fret_index) {
    let open_note = strings[string_index].toUpperCase();
    let base = notes.value.indexOf(open_note);

    if (base === -1) base = 0;

    let offset = fret_index;
    let note_index = (base + offset) % notes.value.length;
    return notes.value[note_index];
}

function should_show_note(string_index, fret_index) {
    if (!current_scale_data.value) return false;
    const note = get_note(string_index, fret_index);

    return scale_note_set.value.has(note);
}

const degree_map = computed(() => {
    const map = {};
    const scale = current_scale_data.value;
    if (!scale) return map;
    const formulaItems = scale.formula.split(' ');
    let root_idx = notes.value.indexOf(current_note.value);
    if (root_idx === -1) return map;

    let current_idx = root_idx;
    for (let i = 0; i < scale.intervals.length; i++) {
        current_idx = (current_idx + scale.intervals[i]) % 12;
        map[notes.value[current_idx]] = formulaItems[i + 1] || '';
    }
    map[notes.value[root_idx]] = formulaItems[0] || '';

    console.log(current_scale_data.value);
    
    return map;
});

function display_note(string_index, fret_index) {
    const note = get_note(string_index, fret_index);
    if (use_degree.value && degree_map.value[note]) {
        return degree_map.value[note];
    }
    return note;
}

function is_root_note(string_index, fret_index) {
    const note = get_note(string_index, fret_index);
    return note === current_note.value;
}

function select_note(note) {
    current_note.value = note;
}

function sharp_to_flat(note) {
    const index = notes_sharp.indexOf(note);
    if (index === -1) {
        return note;
    }

    return notes_flat[index];
}

function flat_to_sharp(note) {
    const index = notes_flat.indexOf(note);
    if (index === -1) {
        return note;
    }

    return notes_sharp[index];
}

const categories = {
    major: {
        label: 'Major',
        types: {
            major_scale: { name: 'Major Scale', type: 'scale', intervals: [2, 2, 1, 2, 2, 2, 1], formula: '1 2 3 4 5 6 7' },
            maj7: { name: 'Major 7th', type: 'arpeggio', intervals: [4, 3, 4, 1], formula: '1 3 5 7' },
            maj9: { name: 'Major 9th', type: 'arpeggio', intervals: [4, 3, 4, 3, 2], formula: '1 3 5 7 9' },
            maj_triad: { name: 'Major Triad', type: 'arpeggio', intervals: [4, 3, 5], formula: '1 3 5' }
        }
    },
    minor: {
        label: 'Minor',
        types: {
            natural_minor: { name: 'Natural Minor', type: 'scale', intervals: [2, 1, 2, 2, 1, 2, 2], formula: '1 2 ♭3 4 5 ♭6 ♭7' },
            harmonic_minor: { name: 'Harmonic Minor', type: 'scale', intervals: [2, 1, 2, 2, 1, 3, 1], formula: '1 2 ♭3 4 5 ♭6 7' },
            melodic_minor: { name: 'Melodic Minor', type: 'scale', intervals: [2, 1, 2, 2, 2, 2, 1], formula: '1 2 ♭3 4 5 6 7' },
            m7: { name: 'Minor 7th', type: 'arpeggio', intervals: [3, 4, 3, 2], formula: '1 ♭3 5 ♭7' },
            m9: { name: 'Minor 9th', type: 'arpeggio', intervals: [3, 4, 3, 4, 2], formula: '1 ♭3 5 ♭7 9' },
            min_triad: { name: 'Minor Triad', type: 'arpeggio', intervals: [3, 4, 5], formula: '1 ♭3 5' }
        }
    },
    pentatonic: {
        label: 'Pentatonic',
        types: {
            major_pentatonic: { name: 'Major Pentatonic', type: 'scale', intervals: [2, 2, 3, 2, 3], formula: '1 2 3 5 6' },
            minor_pentatonic: { name: 'Minor Pentatonic', type: 'scale', intervals: [3, 2, 2, 3, 2], formula: '1 ♭3 4 5 ♭7' }
        }
    },
    modes: {
        label: 'Modes',
        types: {
            ionian: { name: 'Ionian', type: 'scale', intervals: [2, 2, 1, 2, 2, 2, 1], formula: '1 2 3 4 5 6 7' },
            dorian: { name: 'Dorian', type: 'scale', intervals: [2, 1, 2, 2, 2, 1, 2], formula: '1 2 ♭3 4 5 6 ♭7' },
            phrygian: { name: 'Phrygian', type: 'scale', intervals: [1, 2, 2, 2, 1, 2, 2], formula: '1 ♭2 ♭3 4 5 ♭6 ♭7' },
            lydian: { name: 'Lydian', type: 'scale', intervals: [2, 2, 2, 1, 2, 2, 1], formula: '1 2 3 ♯4 5 6 7' },
            mixolydian: { name: 'Mixolydian', type: 'scale', intervals: [2, 2, 1, 2, 2, 1, 2], formula: '1 2 3 4 5 6 ♭7' },
            locrian: { name: 'Locrian', type: 'scale', intervals: [1, 2, 2, 1, 2, 2, 2], formula: '1 ♭2 ♭3 4 ♭5 ♭6 ♭7' },
            aeolian: { name: 'Aeolian', type: 'scale', intervals: [2, 1, 2, 2, 1, 2, 2], formula: '1 2 ♭3 4 5 ♭6 ♭7' }
        }
    },
    diminished: {
        label: 'Diminished',
        types: {
            dim_triad: { name: 'Diminished Triad', type: 'arpeggio', intervals: [3, 3, 6], formula: '1 ♭3 ♭5' },
            m7b5: { name: 'Half-Diminished 7th', type: 'arpeggio', intervals: [3, 3, 4, 2], formula: '1 ♭3 ♭5 ♭7' },
            dim7: { name: 'Diminished 7th', type: 'arpeggio', intervals: [3, 3, 3, 3], formula: '1 ♭3 ♭5 ♭♭7' }
        }
    },
    augmented: {
        label: 'Augmented',
        types: {
            aug_triad: { name: 'Augmented Triad', type: 'arpeggio', intervals: [4, 4, 4], formula: '1 3 ♯5' },
            aug7: { name: 'Augmented 7th', type: 'arpeggio', intervals: [4, 4, 3, 1], formula: '1 3 ♯5 ♭7' }
        }
    },
    blues: {
        label: 'Blues',
        types: {
            blues: { name: 'Blues Scale', type: 'scale', intervals: [3, 2, 1, 1, 3, 2], formula: '1 ♭3 4 ♭5 5 ♭7' }
        }
    },
    other: {
        label: 'Other',
        types: {
            chromatic: { name: 'Chromatic', type: 'scale', intervals: [1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1], formula: 'All 12 Notes' },
            dom7: { name: 'Dominant 7th', type: 'arpeggio', intervals: [4, 3, 3, 2], formula: '1 3 5 ♭7' },
            dom9: { name: 'Dominant 9th', type: 'arpeggio', intervals: [4, 3, 3, 4, 2], formula: '1 3 5 ♭7 9' },
            sus4: { name: 'Sus4', type: 'arpeggio', intervals: [5, 2, 5], formula: '1 4 5' },
            sus2: { name: 'Sus2', type: 'arpeggio', intervals: [2, 5, 5], formula: '1 2 5' }
        }
    }
};
</script>
<template>
    <div class="content" :style="{ '--accent': accent_color }">
        <div class="fretbroad">
            <div class="strings" v-for="(string, string_index) in strings" :key="string_index">
                <div class="fret" v-for="fret in frets" :key="fret" :class="{ openstring: (fret) === 0 }">
                    <div class="fretnumber" v-if="string_index === 0 && fret !== 0">{{ fret }}</div>
                    <div class="fretmarker" v-if="string_index === 5 && has_marker(fret)">
                        <div></div>
                        <div v-if="fret === 12"></div>
                    </div>
                    <div class="string">
                        <div v-if="should_show_note(string_index, fret)" class="note"
                            :class="{ root: is_root_note(string_index, fret) }">{{ display_note(string_index, fret) }}
                        </div>
                    </div>
                </div>
            </div>
        </div>

        <div class="infomation">
            <div class="name">
                {{ current_note }} {{ current_scale_data?.name }}
            </div>
            <div class="group">
                <div class="group1">
                    <span class="label">Note:</span><span v-for="note in scale_note_set" :key="note">{{ note.trim()
                        }}</span>
                </div>
                <div class="group1">
                    <span class="label">Formula:</span><span v-for="form in current_scale_data?.formula.split(' ')"
                        :key="form">{{ form.trim() }}</span>
                </div>
                <div class="group1">
                    <span class="label">Intervals:</span><span v-for="form in current_scale_data?.intervals"
                        :key="form">{{ form }}</span>
                </div>
            </div>
            <div class="group scale-type px-4 py-2 rounded-lg font-medium border flex items-center text-center uppercase"
                :class="{ 'bg-purple-500/20 text-purple-400 border-purple-500': current_scale_data?.type == 'arpeggio', 'bg-sky-500/20 text-sky-400 border-sky-500': current_scale_data?.type != 'arpeggio' }">
                {{ current_scale_data?.type }}</div>
        </div>

        <div class="controls">
            <div class="control-group" style="max-width: 440px;">
                <label>Root Note</label>
                <div class="note-option">
                    <button v-for="note in notes" :key="note" :class="{ active: note === current_note }"
                        @click="select_note(note)" class="note-button">
                        {{ note }}
                    </button>
                </div>
            </div>

            <div class="control-group" style="min-width: 200px;">
                <label>Category</label>
                <select v-model="selected_category_key" class="select-input">
                    <option v-for="(cat, key) in categories" :key="key" :value="key">
                        {{ cat.label }}
                    </option>
                </select>
                <label>Scale / Arpeggio</label>
                <select v-model="selected_type_key" class="select-input">
                    <option v-for="(type, key) in current_category_types" :key="key" :value="key">
                        {{ type.name }}
                    </option>
                </select>
            </div>

            <div class="control-group">
                <label>Use Degree</label>
                <label class="toggle-switch">
                    <input type="checkbox" :checked="use_degree" v-model="use_degree" class="toggle-input" />
                    <span class="toggle-slider"></span>
                    <span class="toggle-labels">
                        <span class="toggle-label-on">On</span>
                        <span class="toggle-label-off">Off</span>
                    </span>
                </label>

                <label>Use Flat</label>
                <label class="toggle-switch">
                    <input type="checkbox" :checked="use_flat" v-model="use_flat" class="toggle-input" />
                    <span class="toggle-slider"></span>
                    <span class="toggle-labels">
                        <span class="toggle-label-on">On</span>
                        <span class="toggle-label-off">Off</span>
                    </span>
                </label>
            </div>

            <div class="control-group">
                <label>Fret Start</label>
                <Numeric :min="0" :max="17" v-model="fret_start" />
                <label>Fret End</label>
                <Numeric :min="5" :max="21" v-model="fret_end" />
            </div>

            <div class="control-group">
                <label>Accent Color</label>
                <div class="color-presets">
                    <button v-for="color in presetColors" :key="color" :style="{ background: color }"
                        :class="{ active: accent_color === color }" @click="accent_color = color"
                        class="preset-color" />
                    <input type="color" v-model="accent_color" class="color-input" />
                </div>
            </div>
        </div>
    </div>
</template>
<style scoped>
.content {
    background: #1a1a1a;
    min-width: 100vw;
    min-height: 100vh;
    padding: 16px;
}

.fretbroad {
    margin-top: 20px;
    margin-bottom: 48px;
}

.strings {
    display: flex;
    justify-content: space-between;
}

.string {
    position: relative;
    border: 1px solid #fff;
    width: 100%;
    display: flex;
    justify-content: center;
    align-items: center;
}

.fret {
    position: relative;
    border-right: 1px solid #fff;
    height: 50px;
    width: 100%;
    display: flex;
    justify-content: center;
    align-items: center;
}

.fretnumber {
    position: absolute;
    text-align: center;
    top: -28px;
    font-size: 1rem;
    font-weight: 600;
    opacity: 0.7;
}

.fretmarker {
    position: absolute;
    display: flex;
    gap: 4px;
    top: 56px;
}

.fretmarker div {
    width: 18px;
    aspect-ratio: 1;
    border-radius: 100%;
    background: #fff;
    opacity: 0.7;
}

.openstring {
    border-right: 3px solid #fff;
}

.note {
    position: absolute;
    width: 30px;
    height: 30px;
    border-radius: 100%;
    background: var(--accent);
    display: flex;
    text-align: center;
    align-items: center;
    justify-content: center;
    font-weight: 500;
}

.note-option {
    display: flex;
    gap: 12px;
    flex-wrap: wrap;
}

.controls {
    background: #2a2a2a;
    padding: 20px;
    border-radius: 12px;
    display: flex;
    justify-content: center;
    gap: 32px;
    margin-bottom: 30px;
    width: fit-content;
    margin-left: auto;
    margin-right: auto;
    box-shadow: 0 4px 12px rgba(0, 0, 0, 0.3);
}

.control-group {
    display: flex;
    flex-direction: column;
    gap: 8px;
}

.control-group label {
    font-size: 0.9rem;
    color: #ccc;
    font-weight: 600;
    text-transform: uppercase;
    letter-spacing: 1px;
}

@media (min-width: 768px) {
    .controls {
        flex-direction: row;
        flex-wrap: wrap;
        align-items: flex-start;
    }

    .control-group {
        min-width: 150px;
    }
}

.select-input {
    padding: 10px;
    background: #333;
    border: 1px solid #444;
    color: #fff;
    border-radius: 6px;
    font-size: 1rem;
    cursor: pointer;
}

.note-button {
    flex: 1 1 60px;
    max-width: 60px;
    text-align: center;
    white-space: nowrap;
    padding: 8px 16px;
    background: #333;
    border: 1px solid #555;
    color: #fff;
    border-radius: 6px;
    cursor: pointer;
    font-weight: 500;
    transition: all 0.2s ease;
}

.note-button:hover {
    background: #444;
    border-color: #777;
}

.note-button.active {
    background: var(--accent);
    border-color: var(--accent);
    box-shadow: 0 0 3px var(--accent);
}

.root {
    background: #fff;
    color: var(--accent);
}

.infomation {
    margin-bottom: 20px;
    display: flex;
    justify-content: center;
    align-items: center;
    flex: 1 auto;
}

.infomation .name {
    font-size: 1.75rem;
    font-weight: 600;
    color: #fff;
    text-align: center;
}

.infomation .group {
    display: flex;
    margin-left: 38px;
    flex-direction: column;
}

.group1 {
    display: flex;
    gap: 16px;
}

.group1 span {
    display: inline-block;
    min-width: 25px;
    color: #fff;
}

.group1 span.label {
    display: inline-block;
    min-width: 65px;
    color: #fff;
    font-size: 0.9em;
}

.toggle-switch {
    margin-top: 6px;
    margin-bottom: 7px;
    position: relative;
    display: inline-flex;
    align-items: center;
    cursor: pointer;
    user-select: none;
}

.toggle-input {
    position: absolute;
    opacity: 0;
    width: 0;
    height: 0;
}

.toggle-slider {
    position: relative;
    width: 60px;
    height: 32px;
    background: #333;
    border-radius: 32px;
    transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);
    box-shadow: inset 0 2px 4px rgba(0, 0, 0, 0.3);
}

.toggle-slider::before {
    content: '';
    position: absolute;
    width: 26px;
    height: 26px;
    left: 3px;
    top: 3px;
    background: #fff;
    border-radius: 50%;
    transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);
    box-shadow: 0 2px 6px rgba(0, 0, 0, 0.3);
}

.toggle-input:checked+.toggle-slider {
    background: var(--accent);
}

.toggle-input:checked+.toggle-slider::before {
    transform: translateX(28px);
    background: #fff;
}

.toggle-labels {
    margin-left: 12px;
    font-size: 14px;
    font-weight: 500;
    color: #ccc;
    display: flex;
    gap: 8px;
}

.toggle-label-on {
    opacity: 0.4;
    transition: opacity 0.3s;
}

.toggle-label-off {
    opacity: 1;
    transition: opacity 0.3s;
}

.toggle-input:checked~.toggle-labels .toggle-label-on {
    opacity: 1;
}

.toggle-input:checked~.toggle-labels .toggle-label-off {
    opacity: 0.4;
}

.color-presets {
    display: flex;
    gap: 8px;
    align-items: center;
    flex-wrap: wrap;
}

.preset-color {
    width: 36px;
    height: 36px;
    border-radius: 8px;
    border: 2px solid transparent;
    cursor: pointer;
    transition: all 0.2s ease;
}

.preset-color:hover {
    transform: scale(1.1);
    border-color: #fff;
}

.preset-color.active {
    border-color: #fff;
    box-shadow: 0 0 0 3px rgba(255, 255, 255, 0.3);
}

.color-input {
    width: 50px;
    height: 43px;
    border: 2px solid #555;
    border-radius: 8px;
    cursor: pointer;
    background: var(--accent);
}

input[type="color"] {
    border: 2px solid #555;
    -webkit-appearance: none;
    appearance: none;
}

input[type="color"]::-webkit-color-swatch-wrapper {
    padding: 0;
}

input[type="color"]::-webkit-color-swatch {
    border: none;
}

input[type="color"]:focus {
    border: 2px solid #fff;
}
</style>