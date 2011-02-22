Summary
=======

The beginnings of a node-based REST proxy as an implementation of the [Gateway pattern](http://martinfowler.com/eaaCatalog/gateway.html).

The goal is to support parallelized and/or sequential REST calls, and to extract pieces of the downstream responses into an aggregate object suitable for a response.

Target Interface
================

recipe = Proxie.newRecipe();
recipe.against.get("/entity/{id}/child/{id2}")
	.do
	.inParallel(
		Proxie.get("http://entities.me.com/{id}")
			.acceptJSON
			.extract("entity", function(result) {
				return result.data;	
			}),
		Proxie.get("http://children.me.com/{id2}")
			.acceptXML
			.extract("child", function(result) {
				return result.data;
			})
	).andWaitForAll
	then(
		Proxie.post("http://localhost/authorize").withFormBody({
				"CHILD-ID" : Proxie.flash("child").resourceId,
				"USER-ID" : Proxie.requestHeader("X-AUTHENTICATION")
			})
			.acceptJSON
			.abortIf(function(result) {
				return result.status != "success";
			})
	)
	.return({
		entity: Proxie.flash("entity"),
		child: Proxie.flash("child")
	})
	.asJSEND;
