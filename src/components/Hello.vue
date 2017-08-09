<template>
  <div>
  	<div>
    	<button @click="add">Add</button>
  	</div>

  	<div>
  		<input v-model="search" @keyup.enter="find">
  		{{fl}}
  	</div>

		<li v-for="item in items">
			{{item[0]}} is {{item[1]}} ({{item[2]}})
		</li>
  </div>
</template>


<script>
import ds from 'datascript';
import Chance from 'chance';
import {createClient} from 'contentful';

const SPACE_ID = "nnlhr3cwksjs";
const ACCESS_TOKEN =
	"f74ca89ce0dae36080afe1026708c555912e65803eb59f46a1032677f992505b";

const client = createClient({
	space: SPACE_ID,
	accessToken: ACCESS_TOKEN
});

// client.getEntry('24DPGBDeGEaYy8ms4Y8QMQ').then(entry => console.log(entry));

var ch = new Chance();

var schema = {
	":p/id": {
		":db/unqiue": ":db.unique/identity"
	},
	":p/job": {
		":db/type": ":db.type/ref"
	}
}

var dbschema = {
	"eid": {
		":db/unqiue": ":db.unique/identity"
	},
	":author/blogs": {
		":db/type": ":db.type/ref",
		":db/cardinality": ":db.cardinality/many"
	}
}

var conn = ds.create_conn(schema);
var dbconn = ds.create_conn(dbschema);

ds.transact(conn, [
	{
		":db/id": 1,
		":job/type": "Designer"
	},
	{
		":db/id": 2,
		":job/type": "Developer"
	}
]);

var sample = [
	{
		":db/id": -3,
		":p/id": "7zxnlkb",
	  ":p/name": "Mercy",
	  ":p/job": 1,
	  ":p/age": 19
	},
	{
		":db/id": -4,
  	":p/id": "zg1jp4q",
  	":p/name": "Jack",
  	":p/job": 2,
  	":p/age": 17
  }
]

sample.forEach((s, i) => {
	ds.transact(conn, [s]);
});

var db = ds.db(conn);
var query = `
	[:find ?name ?age ?type
	 :where
	 [?e ":p/name" ?name]
	 [?e ":p/age" ?age]
	 [?e ":p/job" ?c]
	 [?c ":job/type" ?type]
	]
`;

var q2 = `
	[:find ?name ?age ?type
	 :in $ ?name
	 :where
	 [?e ":p/name" ?name]
	 [?e ":p/age" ?age]
	 [?e ":p/job" ?c]
	 [?c ":job/type" ?type]]
`
// ds.q(q2, ds.db(conn), "Mercy")
// ds.entity(ds.db(conn), [":p/id", "7zxnlkb"]).get(":p/name")

// client.getContentTypes()
// .then((response) => console.log(response.items))
// .catch(console.error)


export default {
  name: 'hello',
  beforeCreate: function () {

  	let csFormat = (entry) => {
  		let type = entry.sys.contentType.sys.id;
  		let obj = {
  			type: type,
  			eid: entry.sys.id
  		};

  		Object.keys(entry.fields).forEach(k => {
  			let val = entry.fields[k]['en-US'];
  			if (Array.isArray(val)) {
	  			obj[`${type}/${k}`] = entry.fields[k]['en-US'].map(e => e.sys.id);
  			} else {
	  			obj[`${type}/${k}`] = entry.fields[k]['en-US'];
	  		}
  		})

  		// console.log(obj)
  		ds.transact(dbconn, [obj]);
  	}

  	client.sync({
		  initial: true
		})
		.then((response) => {
	
			response.entries.forEach((e, i) => csFormat(e, i));
			console.log(
				ds.q(`[:find ?name ?blogs
						 	:where
						 	[?e "eid" "6ieGakZRoA6y4ioOEY6G4s"]
							[?e "author/name" ?name]
							[?e "author/blogs" ?blogs]
						 	]`, ds.db(dbconn)));

		}).catch(console.error);
  },

  data () {
    return {
    	db: ds.db(conn),
    	fl: [],
    	search: ""
    }
  },
  methods: {
  	add: function (e) {
  		let eid = (this.items.length + 1) * (-1);

  		ds.transact(conn, [{
  			":db/id": eid,
  			":p/name": ch.name(),
  			":p/age": ch.age(),
  			":p/id": Math.random().toString(36).substr(2, 7),
  			":p/job": 1
  		}]);

  		this.db = ds.db(conn)
  	},
  	find: function (e) {
  		this.fl = ds.q(q2, this.db, this.search);
  	}
  },
  computed: {
  	items: function () {
  		return ds.q(query, this.db);
  	},
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
</style>
