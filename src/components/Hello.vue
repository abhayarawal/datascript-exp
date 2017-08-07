<template>
  <div>
    {{list}}

    <div>
    	<button @click="add">Add</button>
  	</div>
  </div>
</template>


<script>
import ds from 'datascript';
import Chance from 'chance';

var ch = new Chance();

var schema = {
	":p/id": {
		":db/unqiue": ":db.unique/identity"
	}
}

var conn = ds.create_conn(schema);

var sample = [
	{
		":db/id": -1,
		":p/id": "7zxnlkb",
	  "name": ch.name(),
	  "age": 19
	},
	{
		":db/id": -2,
  	":p/id": "zg1jp4q",
  	"name": ch.name(),
  	"age": 17
  }
]

sample.forEach((s, i) => {
	ds.transact(conn, [s]);
});

var db = ds.db(conn);

export default {
  name: 'hello',
  data () {
    return {
    	list: ds.q(`
				[:find ?name ?age
				 :where
				 [?e "name" ?name]
				 [?e "age" ?age]
				]
				`, db)
    }
  },
  methods: {
  	add: function (e) {
  		let eid = (this.list.length + 1) * (-1);
  		ds.transact(conn, [
  			[":db/add", eid, "name", ch.name()],
  			[":db/add", eid, "age", ch.age()],
  			[":db/add", eid, ":p/id", Math.random().toString(36).substr(2, 7)]
  		]);

		  this.list = ds.q(`
				[:find ?name ?age
				 :where
				 [?e "name" ?name]
				 [?e "age" ?age]
				]
				`, ds.db(conn));
  	}
  },
  computed: {
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
</style>
