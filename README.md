DNF (Dynamic Network Format)
============================

DNF is a format for representing static and dynamic networks.

Author: *Francesco Ficarola*

Versions
--------

* Draft v0.1.0: versions/dnf-v0.1.0.md

Warning: this spec is still under definition. Until it's marked as 1.0.0, you should to consider it as an unstable version.

Objectives
----------

DNF aims to be a minimal representation format for static and dynamic networks. It should be easy to read or parse due to its clear syntax. DNF is designed to efficiently store all information among nodes and edges of a graph. Furthermore, DNF should be easy to learn, read and parse into data structures in a wide variety of programming languages.

How does it work?
-----------------

Traditional representations of time-aggregated graphs use, for each node or edge, a sequence of timestamps (single or continuous), e.g. GEXF file format use the concept of spells:

	<node id="1001">
	<spells>
	<spell start="1335090239" end="1335090241" />
	<spell start="1335090242" end="1335090243" />
	<spell start="1335090244" end="1335090249" />
	</spells>
	</node>

But suppose to have about 100 nodes and 500 edges for a long time of interaction, then you can suppose how the file exponentially increases its size.

The idea behind DNF is very simple: it tries to reduce the file size thanks to gaps among timestamps. For instance, if we have the following dynamics and assume that the global initial timestamp of the dynamic graph is 1335090220, i.e.:

* node 1001 was in the following timestamps: 1335090242, 1335090243, 1335090246, 1335090247, 1335090249

then DNF fixes the initial global timestamps in the header of the file and the initial node gap will be the difference between the initial node timestamp and the global initial timestamp (i.e. `22 = 1335090242 - 1335090220`). Next gaps will be computed with respect to their previous timestamps, e.g. for the second gap, you have 1 = 1335090243 - 1335090242, and for the last gap you have 2 = 1335090249 - 1335090247, so the whole line will be:

	[1001] (22,1,3,1,2)

In addition, DNF manages continuous timestamps to further reduce the file size. Continuous timestamps are represented by `+` symbol followed by the number of continuous instants (excluded the first one), e.g.:

* node 1003 was in the following timestamps: 1335090249, 1335090251, 1335090252, 1335090253, 1335090254, 1335090259

then the corresponding DNF line will be:

	[1003] (29,2,+3,5)

where `29` is the first gap between timestamp 1335090249 and the global initial timestamp, `2` is the gap between timestamp 1335090251 and 1335090249, `+3` is a continuous gap for timestamps 1335090251 (excluded), 1335090252, 1335090253, 1335090254, and the last gap `5` is the difference between 1335090249 and 1335090254. Now, you're probably wondering why the continuous gap is `3` and not `4`. The timestamp 1335090251 is the first of continuous instants, so it's already represented by gap `2`; if we include it into the continuous gap, then we may not know the dimension of the gap between the first instant of continuous gap (1335090251) and the previous one (1335090249).

So, a very important feature of continuous gaps is <u>the exclusion of the first instant of the sequence</u>, in order to manage its gap with the previous element.

Similarly, we have same features for edges, e.g.:

	[1010,1089] (39,1,+7,3,+2,10)

Example 1: A static graph, without attributes neither labels:
-------------------------------------------------------------

	# Graph configuration
	[header]
	graphtype:{static}, defaultedgetype:{undirected}
	nodeattrs:{}, edgeattrs:{}

	# Information about nodes
	[nodes]
	[1001]
	[1002]
	[1003]
	[1004]

	# Information about edges
	[edges]
	[1001,1002]
	[1001,1003] 
	[1002,1004]

Example 2: a static graph, with attributes and labels:
------------------------------------------------------

	# Graph configuration
	[header]
	graphtype:{static}, defaultedgetype:{directed}
	nodeattrs:{label,gender,age}, edgeattrs:{label}

	# Information about nodes
	[nodes]
	[1001] {Bob,M,22}
	[1002] {Melany,F,23}
	[1003] {Mike,M,20}
	[1004] {Alice,F,25}

	# Information about edges
	[edges]
	[1001>1002] {Bob_Melany}
	[1001>1003] {Bob_Mike}
	[1002>1004] {Melany_Alice}

Example 3: a dynamic graph with directed and undirected edges
---------------------------------------------------------------

	# Graph configuration
	[header]
	graphtype:{dynamic}, defaultedgetype:{mixed}
	dynamics:{timetype=custom,start=0}
	nodeattrs:{}, edgeattrs:{}

	# Information about nodes
	[nodes]
	[1001] (10,+3,2,+4)
	[1002] (11,+2,31) 
	[1003] (9,+10,2)
	[1004] (12,1,31,+2)

	# Information about edges
	[edges]
	[1001>1002] (11,1) 
	[1001,1003] (10,+3)
	[1002>1004] (12,1,31)

Example 4: a dynamic graph with attributes and "weight"
----------------------------------------------------------------------------------------------

	# Graph configuration
	[header]
	graphtype:{dynamic}, defaultedgetype:{undirected}
	dynamics:{timetype=timestamp,start=1318836335}
	nodeattrs:{gender,age}, edgeattrs:{weight}

	# Information about nodes
	[nodes]
	[1001] {M,22} (10,+3,2,+4)
	[1002] {F,23} (11,+2,31) 
	[1003] {M,20} (9,+10,2)
	[1004] {F,25} (12,1,31,+2)

	# Information about edges
	[edges]
	[1001,1002] {2} (11,1) 
	[1001,1003] {2} (10,+3)
	[1002,1004] {3} (12,1,31)

Specification
-------------

* DNF is case sensitive
* Comments in DNF starts with the hash character, `#`, and extend to the end of the physical line
* Whitespaces or blank lines are not considered

Syntax
------

Mainly, a DNF file is formed by three sections: header, nodes and edges. Each section is defined as follows:

************
** HEADER **

	[header]

In the header section you can put all information about graph, dynamics and attributes of your nodes and edges.

*[first line]* Here you'll write the graph type (static or dynamic) and the default edge type (undirected or directed), according the following syntax:

	graphtype:{static | dynamic}, defaultedgetype:{undirected | directed | mixed}

*[second line]* You can skip this line if your graph is **static**. Vice versa, if you're representing a dynamic graph, then in this line you can write information about dynamics, according the following syntax:

	dynamics:{timetype=value,start=value,end=value,timeunit=value}

where the `timetype` and `start` attributes are mandatory, whereas the `end` attribute can be omitted.

If you don't have a specific "start" time, you can simply write: `start=0`. The attribute `timetype` explicits what "type of time" you want to use. At present it supports: `timestamp, datetime or custom`. The first one is defined as the UNIX time: http://en.wikipedia.org/wiki/Unix_time. Datetimes are ISO 8601 dates, but only the full zulu form is allowed. In the end, the custom type could be whatever you define in your software.

In addition, you may use the **timeunit** attribute (not mandatory) to indicate how long is each gap. For instance, if you have a `timeunit=5`, then a gap will be far from its previous gap five units of time. With regard to the continuous gaps, you will have "**timeunit_value**" times the continuous gap, e.g., +3 will be 15. If you simply don't express the timeunit attribute, then the default time unit is equal to **1**.

*[third line]* Here you can write any attributes about your nodes or edges, according the following syntax:

	nodeattrs:{label,attr1,attr2,...}, edgeattrs:{label,weight,attr1,attr2,...}

If your labels are equal to node or edge IDs, then you can omit them. <u>All attributes should be typed and considered as string.</u>

***********
** NODES **

	[nodes]

In this section you'll write all information about nodes of your graph, according to the following syntax:

	[nodeID] {label_value, attr1_value,attr2_value,...} (gap1,gap2,gap3,...) 

where **label_value** is present <u>if and only if in the first element of **nodeattrs** you do explicit **label**</u>, otherwise the line will be:

	[nodeID] {attr1_value,attr2_value,...} (gap1,gap2,gap3,...) 

where **attr_values** are present <u>if and only if in the **nodeattrs** you do explicit a list of attributes</u>, otherwise the line will be:

	[nodeID] (gap1,gap2,gap3,...) 

where (gap1,gap2,gap3,...) are present <u>if and only if the graph type is dynamic</u>.

***********
** EDGES **

	[edges]

In this section you'll write information about all undirected edges of your graph, according to the following syntax:

	[sourceID,targetID] {label_value,weight_value,attr1_value,attr2_value,...} (gap1,gap2,gap3,...)

or directed edges of your graph, according to the following syntax:

	[sourceID>targetID] {label_value,weight_value,attr1_value,attr2_value,...} (gap1,gap2,gap3,...)

where for undirected edges, the two nodes will be separate by `,` while for directed edges you can explicit the direction by using `>`: the first node will be the source, whereas the second node will be the target;

**label_value** is present <u>if and only if in the first element of **edgeattrs** you do explicit *label*</u>, otherwise the line will be:

	[sourceID,targetID] {weight_value,attr1_value,attr2_value,...} (gap1,gap2,gap3,...)

where **weight_value** is present <u>if and only if in the **edgeattrs** you do explicit *weight*</u>, otherwise the line will be:

	[sourceID,targetID] {attr1_value,attr2_value,...} (gap1,gap2,gap3,...)

where **attr_values** are present <u>if and only if in the **edgeattrs** you do explicit a list of attributes</u>, otherwise the line will be:

	[sourceID,targetID] (gap1,gap2,gap3,...)

where **(gap1,gap2,gap3,...)** are present <u>if and only if the graph type is dynamic</u>.

Why?
----

Because we need an efficient, dense and human-readable format to represent static or dynamic graphs.

What are the alternatives?
--------------------------

Well, you may use JSON or GEXF, but both of them are too verbose. In fact, by using JSON or GEXF to represent your dynamic graphs, you will get very large files. SURE!

Need help?
----------

What a question! Of course. Send a pull request if you write a parser, a validator or if you'd like to suggest an improvement in the format specification. We need your help!
