DNF v0.1.0
==========

DNF is a format for representing static and dynamic networks.

Author: *Francesco Ficarola*

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
