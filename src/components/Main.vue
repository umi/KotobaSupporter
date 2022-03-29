<script>
import _ from 'lodash'
import dayjs from 'dayjs'
import WordButton from './WordButton.vue'
import Loading from './Loading.vue'
import { onMounted, ref, computed } from 'vue'

export default {
	components: {
		WordButton,
		Loading
	},
	setup() {
		const list = ref([])
		const ansList = ref([])
		const ansInput = ref(null)
		const searchList = ref([])
		const searchInput = ref(null)
		const inputList = ref([])
		const inputErr = ref(false)
		const loadingAns = ref(false)
		const loadingSearch = ref(false)

		const CLOSE_STATUS_KANA = {
			'a0': 'あぁ',
			'a1': 'いぃ',
			'a2': 'うゔぅ',
			'a3': 'えぇ',
			'a4': 'おぉ',
			'a5': 'やゃ',
			'a6': 'ゆゅ',
			'a7': 'よょ',
			'a8': 'かが',
			'a9': 'きぎ',
			'a10': 'くぐ',
			'a11': 'けげ',
			'a12': 'こご',
			'a13': 'さざ',
			'a14': 'しじ',
			'a15': 'すず',
			'a16': 'せぜ',
			'a17': 'そぞ',
			'a18': 'ただ',
			'a19': 'ちぢ',
			'a20': 'つづっ',
			'a21': 'てで',
			'a22': 'とど',
			'a23': 'はばぱ',
			'a24': 'ひびぴ',
			'a25': 'ふぶぷ',
			'a26': 'へべぺ',
			'a27': 'ほぼぽ'
		}

		const CONSONANT_STATUS_KANA = {
			'b0': 'あいうえおぁぃぅぇぉ',
			'b1': 'かきくけこがぎぐげご',
			'b2': 'さしすせそざじずぜぞ',
			'b3': 'たちつてとだぢづでどっ',
			'b4': 'なにぬねの',
			'b5': 'はひふへほばびぶべぼぱぴぷぺぽ',
			'b6': 'まみむめも',
			'b7': 'やゆよゃゅょ',
			'b8': 'らりるれろ',
			'b9': 'わを'
		}

		const VOWEL_STATUS_KANA = {
			'c0': 'ぁあかさたなはまやらわがざだばぱゃ',
			'c1': 'ぃいきしちにひみりぎじぢびぴ',
			'c2': 'ぅうくすつぬふむゆるゔぐずづっぶぷゅ',
			'c3': 'ぇえけせてねへめれげぜでべぺ',
			'c4': 'ぉおこそとのほもよろをごぞどぼぽょ'
		}

		const getData = () => {
			const url = 'https://raw.githubusercontent.com/taximanli/kotobade-asobou/main/src/constants/validGuesses.ts?ver=' + dayjs().format('YYYYMMDD')
			fetch(url)
				.then(res => {
					return res.text()
				})
				.then(data => {
					list.value = _.uniq(_.transform(data.split(/\r\n|\n/), (result, value) => {
						if(value.indexOf("'") === 0) {
							result.push(value.replace(/'|,/g, ''))
						}
					}, []))
				})
		}

		const k2h = (str) => {
			return str.replace(/[\u30a1-\u30f6]/g, function(match) {
				var chr = match.charCodeAt(0) - 0x60
				return String.fromCharCode(chr)
			})
		}

		const showInputErr = () => {
			inputErr.value = true
			setTimeout(() => {inputErr.value = false}, 3000)
		}

		const registItem = () => {
			// console.log(ansInput.value.value)
			if(typeof ansInput.value.value === 'string' && ansInput.value.value.length === 4){
				const registStr = k2h(ansInput.value.value)
				if(_.indexOf(list.value, registStr) > -1){
					inputList.value.push([registStr, [0,0,0,0]])
					ansInput.value.value = ''
					return
				}
			}

			showInputErr()
		}

		const deleteItem = (index) => {
			inputList.value.splice(index, 1)
		}

		const changeItem = (index, num) => {
			if(_.isArray(inputList.value[index]) && _.isArray(inputList.value[index][1])){
				inputList.value[index][1][num] = (inputList.value[index][1][num] + 1) % 6
			}
		}

		const getStatusSet = _.memoize((char) => {
			return _.transform([CLOSE_STATUS_KANA, CONSONANT_STATUS_KANA, VOWEL_STATUS_KANA], (res, list) => {
				res.push(_.findKey(list, (o) => {
					return o.indexOf(char) !== -1
				}))
			}, [])
		})

		const indexList = computed(() => {
			const indexList = _.transform(list.value, (res, val, key) => {
				res[4].push(key)
				const words = val.split('')
				_.forEach(words, (v, k) => {
					_.forEach(_.compact(getStatusSet(v)), (set) => {
						if(typeof res[6][k][set] === 'undefined'){ res[6][k][set] = [] }
						res[6][k][set].push(key)
					})
					if(typeof res[k][v] === 'undefined'){ res[k][v] = [] }
					res[k][v].push(key)
					if(_.indexOf(words, v) === k){
						const picks = _.filter(words, o => o === v)
						_.forEach(picks, (char, index) => {
							const word = char.repeat(index + 1)
							if(typeof res[5][word] === 'undefined'){ res[5][word] = [] }
							res[5][word].push(key)
						})
					}
				})
			}, [{}, {}, {}, {}, [], {}, [{}, {}, {}, {}]])
			// console.log(list)

			return indexList
		})

		const inputObj = computed(() => {

			const comp = (o, n) => {
								if(o === 2 || n === 2) return 2
								if(o === 1 || n === 1) return 1
								if(o === 3 || n === 3) return 3
								if(o === 4 || n === 4) return 4
								if(o === 5 || n === 5) return 5
								if(o === 0 || n === 0) return 0 
								return -1 
							}

			return _.transform(inputList.value, (result, value) => {
				const words = value[0].split('')
				_.forEach(words, (val, key) => {
					if(_.indexOf(words, val) === key){
						let picks = _.map(words, (v, k) => {
							return v === val ? value[1][k]: null
						})
						console.log(picks)
						const groups = _.groupBy(picks)
						console.log(groups)
						const len = [1, 0]
						const hit = (groups[1] ? groups[1].length: 0) + (groups[2] ? groups[2].length: 0)
						const hitAll = hit + (groups[3] ? groups[3].length: 0) + (groups[4] ? groups[4].length: 0) + (groups[5] ? groups[5].length: 0)
						if(!groups[null] || words.length - groups[null].length > 0){
							if(hit > 0){
								len[0] = hit
								if(!!groups[0]){
									len[1] = hit
								}
								if(!!groups[1]){
									picks = _.map(picks, v => v === 0 ? 1: v)
								}
							}
						}

						if(!result[val]){
							result[val] = [_.unzipWith([[-1,-1,-1,-1], picks], comp), len]
						}else{
							result[val] = [_.unzipWith([result[val][1], picks], comp), _.unzipWith([result[val][2], len], Math.max)]
						}
					}
				})
			}, {})
		})

		const displayAns = computed(() => {
			return ansList.value.slice(0, 88)
		})
		const displaySearch = computed(() => {
			return searchList.value.slice(0, 100)
		})

		// 候補絞り込み
		const refined = () => {
			loadingAns.value = true
			ansList.value = []
			setTimeout(() => {
				const selectedObj = inputObj.value
				const [allPosList, allDelList] = _.transform(selectedObj, (result, value, key) => {
					const statusSet = getStatusSet(key)
					if(value[1][0] > 1){
						console.log(indexList.value[5][key.repeat(value[1][0])])
						result[0].push(indexList.value[5][key.repeat(value[1][0])])
					}
					if(value[1][1] > 0){
						const reject = indexList.value[5][key.repeat(value[1][1] + 1)]
						console.log(reject)
						if(reject){
							result[1].push(reject)
						}
					}
					const pending = _.indexOf(value[0], 1) > -1
					console.time('for')
					_.forEach(value[0], (v, k) => {
						let matchIndexList = []
						if(v === 0 && _.max(value[0]) === 0){
							result[1].push(indexList.value[5][key])
							if(typeof statusSet[1] !== 'undefined'){
								result[1].push(indexList.value[6][k][statusSet[1]])
							}
							if(typeof statusSet[2] !== 'undefined'){
								result[1].push(indexList.value[6][k][statusSet[2]])
							}
						}else if(v === 2){
							matchIndexList = indexList.value[k][key]
						}else if(v === 3){
							if(typeof statusSet[0] !== 'undefined'){
								matchIndexList = indexList.value[6][k][statusSet[0]].filter(o => indexList.value[k][key].indexOf(o) === -1)
								result[1].push(indexList.value[5][key])
							}
						}else if(v === 4){
							if(typeof statusSet[1] !== 'undefined'){
								matchIndexList = indexList.value[6][k][statusSet[1]].filter(o => indexList.value[k][key].indexOf(o) === -1)
								result[1].push(indexList.value[5][key])
							}
						}else if(v === 5){
							if(typeof statusSet[2] !== 'undefined'){
								matchIndexList = indexList.value[6][k][statusSet[2]].filter(o => indexList.value[k][key].indexOf(o) === -1)
								result[1].push(indexList.value[5][key])
							}
						}
						if(matchIndexList.length > 0){
							result[0].push(matchIndexList)
							result[1].push(indexList.value[4].filter(o => matchIndexList.indexOf(o) === -1))
						}
					})
					console.timeEnd('for')
					const tmpList = _.union(..._.transform(value[0], (res, v, k) => {
						if(v === 1){
							result[1].push(indexList.value[k][key])
						}else if(pending ){
							res.push(indexList.value[k][key])
						}
					}, []))
					if(tmpList.length > 0){
						result[0].push(tmpList)
					}
				}, [[], []])
				console.log(allPosList, allDelList)
				const posList = allPosList.length ? _.intersection(...allPosList): indexList.value[4]
				const delList = _.union(...allDelList)
				console.time('filter')
				const filList = posList.filter(i => delList.indexOf(i) === -1)
				console.timeEnd('filter')
				const num = filList.length
				ansList.value = _.map(filList, (val) => {
					return list.value[val]
				})
				loadingAns.value = false
				// console.log(ansList.value.length, ansList.value)
			}, 0)
		}

		// 検索処理
		const search = () => {
			loadingSearch.value = true
			searchList.value = []
			setTimeout(() => {
				const str = searchInput.value.value
				const searchStrList = _.groupBy(k2h(str).split(''))

				console.time('create mergeIndex')
				const mergeIndex = _.union(..._.transform(searchStrList, (res, val, key) => {
					if(typeof indexList.value[5][key.repeat(val.length)] !== 'undefined'){
						res.push(indexList.value[5][key.repeat(val.length)])
					}
				}, []))
				console.timeEnd('create mergeIndex')

				console.time('create countList')
				const countList = _.transform(mergeIndex, (res, val) => {
					const count = _.reduce(searchStrList, (sum, v, k) => {
						return sum + (indexList.value[5][k.repeat(v.length)].indexOf(val) !== -1 ? v.length: 0)
					}, 0)
					res[val] = {'id': val, 'str': list.value[val], 'count': count, 'len': count}
				}, {})
				console.timeEnd('create countList')

				const sortedList = _.sortBy(countList, (obj) => {return -obj.count})
				searchList.value = sortedList
				loadingSearch.value = false
			}, 0)
			// console.log(sortedList)
		}

		const copyToClipboard = (text) => {
			navigator.clipboard.writeText(text)
				.then(() => {
					console.log(`copied: ${text}`)
				})
				.catch(e => {
					console.error(e)
				})
		}

		onMounted(() => {
			getData()
			ansInput.value.onkeypress = (e) => {
				const key = e.keyCode || e.charCode || 0
				if(key == 13){
					registItem()
				}
			}
			searchInput.value.onkeypress = (e) => {
				const key = e.keyCode || e.charCode || 0
				if(key == 13){
					search()
				}
			}
		})

		return {
			inputObj,
			indexList,
			list,
			ansInput,
			ansList,
			searchInput,
			registItem,
			inputList,
			deleteItem,
			changeItem,
			refined,
			search,
			copyToClipboard,
			loadingAns,
			loadingSearch,
			displaySearch,
			displayAns,
			inputErr
		}
	}
}
</script>

<template>
	<div class="main">
		<div class="answerArea">
			<div class="inputs">
				<input class="input_text" placeholder="回答入力" type="text" maxlength="4" ref="ansInput">
				<button class="input_button" @click="registItem">⏎</button>
			</div>
			<div class="btn">
				<WordButton :inputList="inputList" :delFunc="deleteItem" :changeFunc="changeItem" :refinedFunc="refined" />
			</div>
			<Loading :show="loadingAns" />
			<p>
				候補:{{ ansList.length }}
				/ 全単語:{{ list.length }}
			</p>
			<div class="words">
				<div v-for="item in displayAns" @click="copyToClipboard(item)">{{ item }}</div>
			</div>
		</div>
		<div class="searchArea">
			<div class="inputs">
				<input class="input_text" placeholder="検索文字入力" type="text" ref="searchInput">
				<button class="input_button search" @click="search">
					<svg xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke="currentColor" aria-hidden="true" class="">
						<path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M21 21l-6-6m2-5a7 7 0 11-14 0 7 7 0 0114 0z"></path>
					</svg>
				</button>
			</div>
			<Loading :show="loadingSearch" text="検索中…" />
			<div class="words">
				<div v-for="item in displaySearch" @click="copyToClipboard(item.str)" :class="`count${item.count} len${item.len}`">{{ item.str }}</div>
			</div>
		</div>
	</div>
	<div :class="[inputErr ? 'alert': 'alert non_visi']">
		<div>
			<p id="alert_text">注意<br>辞書内の単語を記入して下さい</p>
		</div>
	</div>
</template>

<style lang="scss" scoped>
.main {
    display: flex;
    justify-content: center;
	gap: 30px;
	padding: 10px 0;
	background: white;

	@media screen and (max-width: 700px) {
		flex-direction: column;
	}
}
.answerArea,
.searchArea {
	min-width: 340px;
}
.answerArea {
	.btn {
		max-width: 340px;
		margin: 0 auto;
	}
}
.inputs {
    display: flex;
    justify-content: center;

	.input_text {
		font-size: 1.5em;
		width: 9em;
		text-align: center;
		border: 0.15em solid #e5e7eb;
		border-radius: 0.25rem;
		padding: 5px;
		/* margin-bottom: 10px; */
		margin: 3px;
	}
	.input_button {
		font-size: 2em;
		margin: 3px;
		border: solid #555 0px;
		background-color: #e5e7eb;
		border-radius: 0.25rem;
		color: black;
		cursor: pointer;
		padding: 0 8px;
		&.search {
			width: 1.4em;
			padding: 5px 9px 0px;
		}
	}
}
.words {
    display: flex;
    flex-wrap: wrap;
    justify-content: space-between;
    gap: 5px 2px;
    max-width: 340px;
    font-size: 1.1em;
    margin: 10px auto 0;

	>div {
		cursor: pointer;
		background: #d1e3c6;
		padding: 2px 6px;
		&.count5 {
			background: #f59494;
		}
		&.count4 {
			background: #fdb36f;
		}
		&.count3 {
			background: #edd164;
		}
		&.count2 {
			background: #c3d373;
		}
	}
}
.alert {
    width: 100vw;
    position: absolute;
    display: flex;
    justify-content: center;

	& div {
		width: 50vw;
		color: white;
		background-color: #f43f5e;
		text-align: center;
		font-weight: bold;
		border-radius: 0.5em;
		margin: 1em;
		padding: 0.4em;
		position: fixed;
		top: 16vw;
	}
}
.non_visi {
	display: none;
}
</style>
