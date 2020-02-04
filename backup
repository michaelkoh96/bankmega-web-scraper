const axios = require('axios')
const $ = require('cheerio')
const Nightmare = require('nightmare')
const fs = require('fs')

const startingURL = 'https://bankmega.com/promolainnya.php'
const CLICK_DELAY = 2000
const SOLUTION_FILE_NAME = './solution.json'

function loadAllCategory(){
	console.log(`Connecting to ${startingURL} ...`)
	return axios.get( startingURL)
	.then((resp) => {
		let categories =  $('#subcatpromo div img' , resp.data).map( (_ , elm) => elm.attribs).get()
		console.log('Connected !')
		return categories
	})
	.catch(err => Promise.reject(new Error(err)))
}

function loadPromoDetail(detailURL){
	return axios.get( detailURL)
	.then((resp) => {
		let detail =  $('.keteranganinside img' , resp.data).attr('src')
		let area =  $('#contentpromolain2 .area b' , resp.data).text()
		let insideTitle =  $('.titleinside h3' , resp.data).text()
		let period =  $('.periode b' , resp.data).map( (_ , elm) => elm).text()

		return {
			det : detail,
			ar : area,
			title : insideTitle,
			per : period
		}
	})
	.catch(err => Promise.reject(new Error(err)))
}

async function loadPromoPerCategory(catID , promoCategory){
	const nightmare = new Nightmare({show : false})
	let selectedID = `#` + catID
	let resObj = {}
	resObj[promoCategory] = {}
	
	// Click element using nightmare
	return await nightmare
	.goto(startingURL)
	.click(selectedID)
	.wait(CLICK_DELAY)
	.evaluate( () => {
		return document.querySelector('#promolain').innerHTML
	})
	.then(res => {
		let promos = []
		$('img' , res).map( (_ , elm) => {
			promos.push({
				title : elm.attribs.title,
				promo_category : catID,
				image_url :`https://www.bankmega.com/${elm.attribs.src}`,
				promo_detail_url :`https://www.bankmega.com/${elm.parent.attribs.href}` 
			})
		})
		resObj[promoCategory] = promos
		return resObj
	})
	.then(async res => {
		let details = [] , area = ""
		for(let j of res[promoCategory]){
			details.push( loadPromoDetail(j.promo_detail_url))
		}

		return Promise.all(details)
		.then(ress => {
			for (let x = 0; x < ress.length ; x++){
				res[promoCategory][x]["detail_inside_title"] = ress[x].title
				res[promoCategory][x]["detail_period"] = ress[x].per
				res[promoCategory][x]["detail_area"] = ress[x].ar
				res[promoCategory][x]["detail_img_url"] =`https://www.bankmega.com/${ress[x].det}` 
			}
			return res
		})
	})
}

 async function startScraping(){
	return loadAllCategory()
	.then(res => {
		console.log('Scraping in process...')
		let promiseArr = []
		for (let i = 0; i < res.length ; i++){
			promiseArr.push(loadPromoPerCategory(res[i].id , res[i].title))
		}
		Promise.all(promiseArr)
		.then( res => {

			return res
		})
		.then( val => {
			fs.writeFile(SOLUTION_FILE_NAME , JSON.stringify(val , null , 4) , err => {
				if (!err){
					
				}
				console.log('Scraping done ! solution.json file has been created')
			})
		})
	})
	.catch(err => console.log(err))
}


startScraping()


