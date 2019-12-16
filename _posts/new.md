When we talking about secure our data, we actually mean three different stages of a data's life. 
Data at rest: When data is sleeping in someone's hard drive.
Data in transit: Data is transferred between two trusted domains. eg. from a database server to compute node, or from your browser to a server.
Data in use: Data is calculated inside a machine (RAM and CPU)
Data in transit problem has been resolved by cryptography, such as https have been widely used. Now a day it is kind of hard to find any public web server without https. If there is, the Chrome browser will jump in between you and the unsafe server and give you a thrill warning. 

Data at rest can be resolved by cryptography as well. Not only for those blob data but also for structural data: Data in the database can be kept encrypted while being indexed. That means you can even query against encrypted data! Magic eh? Just google CryptDB by Raluca Ada Popa and others [link](https://css.csail.mit.edu/cryptdb/). 

Data in use is the hardest one. It is hard because the data need to be computed in CPU it has to be decrypted. I know FHE but it would be too slow to be practical if you are doing some real computation. We can do a query against encrypted data in a database because the query index is relatively easy, we can always find some kind of cryptography algorithm to deal with limited types of query/indexing. But for general-purpose computing, it would be a totally different story. 

How about we step back to review our original question: Why do we need to secure our data? Basically, we do not trust the environment of the storage, transportation and computing environment of our data. Our data is so important to us, and because it can be copied unlimited times as long as disclosed to malicious use. 

The basis of data security is that as long as data leave our fully secured domain it has to be encrypted because technically encrypted data is just garbage without the
key. So we can do either encryption or enclose our data inside the so-called trusted domain - the walled garden. 

Although technically there is no absolute secure domain, generally speaking, we consider something secure as long as we own or we trust even we do not own. Instead of encrypting everything, we can also consider extending the trust to somewhere we do not own, for example, the computing node somewhere in the cloud who is processing my data! 

Oh, no, I know this would be your first response when heard I want you to trust an unknown machine sit in the middle of nowhere. What if the node is just pretent to be a Amazon node, what if the node is owned by Amazon but get hacked, what if some super power coming from government force the owner of the node to disclose my data.... well, there are too many "what-if-s" since we do not trust anything we do not own. Consider this way, even if you own it, can you trust it? Probably not. I know many people have some kind of "smart" home devices at home. Do you know it has the ability to listen to your talking 24 by 7 without your permission? Do you know the small cheap IoT devices usually cannot protect itself efficiently? Not to mention some superpower may generate AI and pretend to be silly when the human approach... Well, I am not going to scare you. I always pull out the power cord of my google home and I am pretty sure it doesn't have a battery in it. 

