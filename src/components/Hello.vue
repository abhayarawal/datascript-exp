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

var ch = new Chance();

var schema = {
	":p/id": {
		":db/unqiue": ":db.unique/identity"
	},
	":p/job": {
		":db/type": ":db.type/ref"
	}
}

var conn = ds.create_conn(schema);

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

export default {
  name: 'hello',
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
