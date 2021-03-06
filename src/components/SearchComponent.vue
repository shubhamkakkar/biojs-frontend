<template>
  <div>
	<b-form inline id="searchForm">
		<b-input class="mb-2 mr-sm-2 mb-sm-0" id="inlineFormInputName2" placeholder="Search for a component" v-model="query" />
	</b-form>
	<div id="results" v-if="isSearching">
		<heading title="Search results" />
		<b-list-group id="resultsContent">
			<router-link v-for="result in queryResults.slice(0, 10)" :key="result" :to='computeURL(result.url_name)' class="link">
				<b-list-group-item class="resultItem">
					<span class="resultName">
						{{result.name}}
					</span>
					<div class="tags">
						Tags:
						<router-link v-for="tag in result.tags" :key="tag" :to="'/search/' + tag" id="tagLinkContainer">
							<span class="resultTag">{{tag}}</span>
						</router-link>
					</div>
				</b-list-group-item>
			</router-link>
			<p id="noResult" v-if="queryResults.length === 0">No result found</p>
		</b-list-group>
	</div>
  </div>
</template>
<script>
/* eslint-disable */
import axios from 'axios';
import fz from 'fuzzaldrin-plus';
import Heading from './Heading.vue';
import {API_URL} from '../DB_CONFIG.js';

export default {
	name: 'SearchComponent',
	introduction: 'Search bar to query across all the components.',
description: `
The SearchComponent component queries across all the components\' names and tags.
A fuzzy search is implemented, more about which can be found here: https://github.com/jeancroy/fuzz-aldrin-plus
### API response for all components:
<img src="https://raw.githubusercontent.com/biojs/biojs-frontend/guide-assets/guide-assets/SearchComponentAPI.png" width="300px" alt="API Response">
`,
token: '',
	data () {
		return {
			query: '',
			components: []
		};
	},
	components: {
		'heading': Heading
	},
	mounted () {
		axios({ method: 'GET', 'url': API_URL + 'all/' }).then(result => {
			this.components = result.data.all_components;
		}, error => {
			console.error(error);
		});
	},
	computed: {
		// get results of the query using Fuzzaldrin Plus plugin. Read more: https://github.com/jeancroy/fuzz-aldrin-plus
		// Read more: https://alligator.io/vuejs/vue-client-side-search/ (Date accessed: June 14, 2018)
		queryResults () {
			// Preparing the query before-hand lets fuzzaldrin-plus optimize things a bit.
			const preparedQuery = fz.prepareQuery(this.query);
			// We use this to keep track of the similarity for each option.
			const scores = {};
			return this.components
			// Score each option & create a new array out of them.
				.map((component, index) => {
					// See how well each component compares to the query.
					const componentScores = [
						component.id,
						component.name,
						// join all the tags and create a string
						component.tags.join(', ')
						// Creating an array of fields and mapping is easier than writing
						// fz.score(...) three times. Same idea.
						// Scores are a non-normalized number
						// representing how similar the query is to the field.

					].map(toScore => fz.score(toScore, this.query, { preparedQuery }));
					// Store the highest score for this option
					// so we can compare it to other options.
					scores[component.id] = Math.max(...componentScores);

					return component;
				})
				// Remove anything with a really low score.
				.filter(component => scores[component.id] > 1)
				// Finally, sort by the highest score.
				.sort((a, b) => scores[b.id] - scores[a.id])
			;
		},
		isSearching () {
			if (this.query.trim() !== '') {
				return true;
			} else {
				return false;
			}
		}
	},
	methods: {
		computeURL (url) {
			return '/component/' + url;
		}
	}
};
</script>
<style lang="scss" scoped>
#searchForm {
  width: 100%;
  display: flex;
  justify-content: center;
  align-items: center;
  margin-bottom: 40px;
  input {
	width: 100%;
  }
}
#results {
	width: 100%;
	display: flex;
	flex-direction: column;
	align-items: center;
	margin-top: -50px;
}
#resultsContent {
	width: 60%;
	margin-bottom: 50px;
	max-height: 300px;
	overflow: auto;
}
.resultName {
	font-weight: bolder;
}
.tags {
	display: flex;
	flex-wrap: wrap;
	align-items: center;
}
.resultTag {
	padding: 1px 5px;
	margin: 0px 5px;
	border-radius: 2px;
	background: #eee;
	margin: 5px;
	transition: 0.2s all ease-in-out;
	color: #000;
	text-decoration: none;
}
.resultTag:hover {
	background: #007E3A;
	color: #fff;
	text-decoration: none;
}
.resultTag:active, .resultTag:visited {
	text-decoration: none;
	color: #000;
}
#tagLinkContainer {
	margin: 5px 0;
}
#noResult {
	font-size: 20px;
	font-weight: bolder;
	text-align: center;
}
.resultItem {
	transition: background 0.2s ease-in-out;
	cursor: pointer;
}
.resultItem:hover {
	background: rgb(230, 230, 230);
}
.link {
	text-decoration: none;
	color: #000;
}
.link:hover {
	text-decoration: none;
	color: #000;
}
@media (max-width: 700px) {
	#searchForm {
		align-items: center;
		input {
		margin-bottom: 0 !important;
		}
	}
	#resultsContent {
		width: 80%;
	}
}
</style>
