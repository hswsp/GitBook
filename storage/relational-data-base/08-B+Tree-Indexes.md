# 08 - B+Tree Indexes

![1.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678268531328-fb40e7fa-f3a4-454b-85e5-1dcc2754bce3.jpeg#averageHue=%231c2534&clientId=uddebc9c7-8495-4&from=ui&id=u0b9b0132&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=582951&status=done&style=none&taskId=u2d052f4e-97ab-4326-945a-d7d87034c80&title=)

![2.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678268529580-8c365242-05c3-4f12-b38b-02438642095d.jpeg#averageHue=%23eeeeee&clientId=uddebc9c7-8495-4&from=ui&id=ub56f56af&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=255208&status=done&style=none&taskId=u8269aeeb-1ea3-4af1-8e2b-6764c01f7cd&title=)

![3.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678268529486-e2dd8963-e2f1-4e36-ac3e-90c6726720e2.jpeg#averageHue=%23efefef&clientId=uddebc9c7-8495-4&from=ui&id=u10277b00&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=198702&status=done&style=none&taskId=ua3e24855-cce8-4222-bd8c-a2d5df4918a&title=)

# Table Indexes

There are a number of different data structures one can use inside of a database system for purposes such as **internal meta-data, core data storage, temporary data structures**, or **table indexes**. For table indexes, which may involve queries with range scans,
A *table index* is a replica of a subset of a table’s columns that is organized and/or sorted for efﬁcient access using a subset of those attributes. So instead of performing a sequential scan, the DBMS can lookup the table index’s auxiliary data structure to ﬁnd tuples more quickly. The DBMS ensures that the contents of the tables and the indexes are always logically in sync.

![4.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678268530077-be80b90d-9a2f-42dd-b056-7010b0a034fd.jpeg#averageHue=%23ededed&clientId=uddebc9c7-8495-4&from=ui&id=u49f2a3aa&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=287757&status=done&style=none&taskId=u417e1558-0a03-4fad-8da8-d6686a95fe5&title=)

There exists a trade-off between the number of indexes to create per database. Although more indexes makes looking up queries faster, indexes also use storage and require maintenance. It is the DBMS’s job to ﬁgure out the best indexes to use to execute queries.

![5.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678268529584-f126763c-76a5-44c1-96b4-4eca5724cc6c.jpeg#averageHue=%23ededed&clientId=uddebc9c7-8495-4&from=ui&id=u4ac33223&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=263345&status=done&style=none&taskId=ucb44053b-a705-4556-b176-6305b434d34&title=)

![6.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678268531283-89756c86-c1b5-4d21-b3ca-341579d03248.jpeg#averageHue=%23f0f0f0&clientId=uddebc9c7-8495-4&from=ui&id=u41fd908c&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=180522&status=done&style=none&taskId=u00053c6f-5e48-4acd-9f5a-675b4815f6a&title=)

# B+Tree

A *B+Tree* is a self-balancing tree data structure that keeps data sorted and allows searches, sequential access, insertion, and deletions in $O(log(n))$. It is optimized for disk-oriented DBMS’s that read/write large blocks of data.

![7.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678268531479-b6ba7c6d-33a9-4c42-879d-4457752c105b.jpeg#averageHue=%23eeeeee&clientId=uddebc9c7-8495-4&from=ui&id=ub653854d&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=256572&status=done&style=none&taskId=u8b4a1433-0fde-4d8e-9370-5d1af76c42f&title=)

![8.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678268532359-28d770a0-f8c5-440e-aa5f-0ad3005a7b5d.jpeg#averageHue=%23e9e9e9&clientId=uddebc9c7-8495-4&from=ui&id=u94ec4bd7&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=601156&status=done&style=none&taskId=u9b4fb3b7-3cb7-4106-8397-079e07bb2e5&title=)

![9.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678268532728-22abfa2d-03a3-433e-ae98-f90ac70d0786.jpeg#averageHue=%23ebebeb&clientId=uddebc9c7-8495-4&from=ui&id=u9d322477&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=515718&status=done&style=none&taskId=uad2e55f9-f83c-4e64-b21a-940948076b7&title=)

Almost every modern DBMS that supports order-preserving indexes uses a B+Tree. There is a speciﬁc data structure called a B-Tree, but people also use the term to generally refer to a class of data structures. The primary difference between the original B-Tree and the B+Tree is that B-Trees stores keys and values in all nodes, while B+ trees store values only in leaf nodes. Modern B+Tree implementations combine features from other B-Tree variants, such as the sibling pointers used in the $B^{link} -Tree$.

![10.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678268533266-f22dbcd6-f80f-45e6-88f4-9434017d166e.jpeg#averageHue=%23ededed&clientId=uddebc9c7-8495-4&from=ui&id=u24e2775e&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=311133&status=done&style=none&taskId=u065e9b6c-426b-487a-b5a5-e3a5ccbccaf&title=)

Formally, a B+Tree is an M-way search tree (where M represents the maximum number of children a node can have) with the following properties:

- It is perfectly balanced (i.e., every leaf node is at the same depth).
- Every **inner node** other than the root is at least half full (M/2 − 1 <= num of keys <= M − 1).
- Every inner node with k keys has k+1 non-null children.

![11.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678268533293-4f00431d-6d43-4fa1-8213-d76c79bad1bc.jpeg#averageHue=%23ededed&clientId=uddebc9c7-8495-4&from=ui&id=uac353efe&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=290587&status=done&style=none&taskId=u8c969818-50ba-49ae-86db-9823bc6506b&title=)

Every node in a B+Tree contains an array of key/value pairs. The keys in these pairs are derived from the attribute(s) that the index is based on. The values will differ based on whether a node is an inner node or a leaf node. For inner nodes, the value array will contain pointers to other nodes. Two approaches for leaf node values are record IDs and tuple data. **Record IDs** refer to a pointer to the location of the tuple. Leaf nodes that have **tuple data** store the the actual contents of the tuple in each node.

![12.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678268533351-7838ca64-b756-4247-9c9d-a9b7012a55e3.jpeg#averageHue=%23edecec&clientId=uddebc9c7-8495-4&from=ui&id=udd81ed4e&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=317849&status=done&style=none&taskId=uc628b1ef-b09e-48c2-8b44-b1969d9970d&title=)

Figure 1: B+ Tree diagram

![13.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678268535898-e0c22566-7887-468e-899b-be5f04f8840d.jpeg#averageHue=%23ededed&clientId=uddebc9c7-8495-4&from=ui&id=u8430adb1&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=290870&status=done&style=none&taskId=u464349b8-0b43-4ac2-9ad7-5ff7101f87f&title=)

Though it is not necessary according to the deﬁnition of the B+Tree, arrays at every node are almost always sorted by the keys.

![14.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678268535902-78d0f823-12c2-44e3-8ace-9b659e949a60.jpeg#averageHue=%23e9e8e8&clientId=uddebc9c7-8495-4&from=ui&id=u2e5cfd8b&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=234846&status=done&style=none&taskId=u42b4d8fa-df4b-4672-8434-a290c4fccf8&title=)

![15.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678268536073-a22f14f1-47b4-47d4-a75f-4d147cb69fd1.jpeg#averageHue=%23e9e9e8&clientId=uddebc9c7-8495-4&from=ui&id=uef8ab285&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=233628&status=done&style=none&taskId=u5f9fc700-4146-4ec1-8b4e-b18d0abf436&title=)

Conceptually, the keys on the inner nodes can be thought of as guide posts. They guide the tree traversal but do not represent the keys (and hence their values) on the leaf nodes. What this means is that you could potentially have a key in an inner node (as a guide post) that is not found on the leaf nodes. Although it must be noted that conventionally inner nodes possess only those keys that are present in the leaf nodes.

![16.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678268537017-7282f622-c0b5-432d-9ec9-19c7c4bde67a.jpeg#averageHue=%23e9e8e8&clientId=uddebc9c7-8495-4&from=ui&id=ubdda432a&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=269120&status=done&style=none&taskId=u6fc53810-29fc-4741-b5ba-5b15e7c9cc3&title=)

![17.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678268537154-2e907f15-3603-4610-ab43-bc7f4de4f9fb.jpeg#averageHue=%23ebe9e9&clientId=uddebc9c7-8495-4&from=ui&id=ue902a42e&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=388818&status=done&style=none&taskId=u23a2276f-9eb8-4727-b4eb-e73d7063727&title=)

![18.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678268537939-0770640f-e042-4dc9-84ae-ec35f42c45a9.jpeg#averageHue=%23ededed&clientId=uddebc9c7-8495-4&from=ui&id=ua7c1153e&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=275485&status=done&style=none&taskId=u6047901e-fda5-4ddf-9f0b-e844977b53e&title=)

# Insertion

To insert a new entry into a B+Tree, one must traverse down the tree and use the inner nodes to ﬁgure out which leaf node to insert the key into.

1. Find correct leaf L.
2. Add new entry into L in sorted order:

- If L has enough space, the operation is done.
- Otherwise split L into two nodes L and L2 .

  - Redistribute entries evenly and copy up middle key.
  - Insert index entry pointing to L2 into parent of L.

1. To split an inner node, redistribute entries evenly, but **push up **the middle key.

![19.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678268538599-632a54b0-6e0c-4715-961b-a80b9eecd813.jpeg#averageHue=%23ececec&clientId=uddebc9c7-8495-4&from=ui&id=u3aef349b&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=330187&status=done&style=none&taskId=u0dd98417-7d97-4072-a6cd-294e9639a11&title=)

[B+ Trees Visualization](https://dichchankinh.com/~galles/visualization/BPlusTree.html)

![20.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678268538074-98729db5-a611-4e04-ac34-5d2c23f8c242.jpeg#averageHue=%23f0efef&clientId=uddebc9c7-8495-4&from=ui&id=u89daa430&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=227594&status=done&style=none&taskId=u69076311-fa6a-4970-b98b-45cc8334907&title=)

# Deletion

Whereas in inserts we occasionally had to split leaves when the tree got too full, if a deletion causes a tree to be less than half-full, we must merge in order to re-balance the tree.

1. Find correct leaf L.
2. Remove the entry:

- If L is at least half full, the operation is done.
- Otherwise, you can try to redistribute, borrowing from sibling.
- If redistribution fails, merge L and sibling.

1. If merge occurred, you must delete entry in parent pointing to L.

![21.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678268539577-05289727-ea32-430e-b007-668c79c3f264.jpeg#averageHue=%23ececec&clientId=uddebc9c7-8495-4&from=ui&id=u29afd195&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=337367&status=done&style=none&taskId=ud1991d28-ca72-4237-be63-e5b8eb707f7&title=)

# Selection Conditions

Because B+Trees are in sorted order, look ups have fast traversal and also do not require the entire key. The DBMS can use a B+Tree index if the query provides any of the attributes of the search key. This differs from a hash index, which requires all attributes in the search key.

![22.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678268540809-073bbcf0-b0e7-40e4-8ad2-d1135dec4dc5.jpeg#averageHue=%23ececec&clientId=uddebc9c7-8495-4&from=ui&id=u1e36b4a8&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=340002&status=done&style=none&taskId=u61ca0e35-3a64-4b1f-8217-29864a9b39d&title=)

## Non-Unique Indexes

Like in hash tables, B+Trees can deal with non-unique indexes by duplicating keys or storing value lists. In the **duplicate keys** approach, the same leaf node layout is used but duplicate keys are stored multiple times. In the **value lists** approach, each key is stored only once and maintains a linked list of unique values.

![23.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678268540845-4e71f4ef-0701-43b9-922a-71bc78aaa6d1.jpeg#averageHue=%23ececec&clientId=uddebc9c7-8495-4&from=ui&id=uceb959d4&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=239852&status=done&style=none&taskId=u7fa86444-160d-4e80-8297-af9ec7dc205&title=)

![24.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678268540951-880387e4-979e-4597-90c0-83aa892f3d30.jpeg#averageHue=%23ececec&clientId=uddebc9c7-8495-4&from=ui&id=u2b212888&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=246851&status=done&style=none&taskId=uebc06875-45fa-45d8-b35d-2ea437922d8&title=)

![25.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678268541040-a48894b8-58fb-4648-8a9a-9afa622046b2.jpeg#averageHue=%23ececec&clientId=uddebc9c7-8495-4&from=ui&id=ufe8a6119&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=258366&status=done&style=none&taskId=u822ce7a6-c6d3-4bef-8904-7e2b8bebba2&title=)

Figure 2: To perform a preﬁx search on a B+Tree, one looks at the ﬁrst attribute on the key, follows the path down and performs a sequential scan across the leaves to ﬁnd all they keys that one wants.

![26.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678268541736-d0b87d92-27b8-4c24-87cf-c13aa6ce71d6.jpeg#averageHue=%23edecec&clientId=uddebc9c7-8495-4&from=ui&id=u3d2010a3&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=299724&status=done&style=none&taskId=uff463387-f3c9-4237-b008-b596f502548&title=)

![27.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678268544045-b074c45a-db2a-4c24-b9c3-483c2e422397.jpeg#averageHue=%23f0efef&clientId=uddebc9c7-8495-4&from=ui&id=u05df3865&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=335268&status=done&style=none&taskId=u0281257d-ff85-4acb-95a2-bcfdeced8b1&title=)

![28.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678268544163-d9472677-2487-44ec-bf24-c6758e3e5e42.jpeg#averageHue=%23f0efef&clientId=uddebc9c7-8495-4&from=ui&id=ubc982156&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=337202&status=done&style=none&taskId=u856e4a9f-6649-4a68-92e6-86b34802f58&title=)

![29.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678268544300-bfe623f9-0fc8-4995-b191-211ec40ef5a2.jpeg#averageHue=%23f0efef&clientId=uddebc9c7-8495-4&from=ui&id=u5a713be3&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=337678&status=done&style=none&taskId=u223c08ef-e30d-46d2-9150-6fad2927cf4&title=)

![30.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678268544416-fae8c7c2-ac99-46da-8942-96145763db64.jpeg#averageHue=%23f0efef&clientId=uddebc9c7-8495-4&from=ui&id=u03c78ba1&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=338071&status=done&style=none&taskId=uc06bcd93-508f-4c96-b75f-df84cda5cbd&title=)

![31.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678268544542-10d96932-dcae-478f-816c-d4e8ee403783.jpeg#averageHue=%23f0efef&clientId=uddebc9c7-8495-4&from=ui&id=u87ab8cf9&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=340009&status=done&style=none&taskId=ua3c316f0-0d9d-4151-a4ef-6974658268b&title=)

![32.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678268547495-b0bf548e-4d7d-4dc7-94ac-e7ed225ed2f8.jpeg#averageHue=%23f0efef&clientId=uddebc9c7-8495-4&from=ui&id=ua0d883ef&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=354595&status=done&style=none&taskId=u2256dfd8-4798-49f7-a2a3-2083553a73a&title=)

## Duplicate Keys

There are two approaches to duplicate keys in a B+Tree.
The ﬁrst approach is to **append record IDs as part of the key.**  Since each tuple’s record ID is unique, this will ensure that all the keys are identiﬁable.
The second approach is to **allow leaf nodes to spill into overﬂow nodes** that contain the duplicate keys. Although no redundant information is stored, this approach is more complex to maintain and modify.

![33.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678268547388-1f2029ca-32eb-49cf-bffa-92e70003b36b.jpeg#averageHue=%23ebebeb&clientId=uddebc9c7-8495-4&from=ui&id=u020508b6&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=332050&status=done&style=none&taskId=u966c7ec3-4b5e-4600-816e-e24e92015bd&title=)

![34.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678268547697-3654890a-fdac-4b0e-899a-1105bd820417.jpeg#averageHue=%23ececec&clientId=uddebc9c7-8495-4&from=ui&id=ua07f7124&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=267475&status=done&style=none&taskId=ube548151-f2f2-45ec-a690-ef3f66fde2d&title=)

![35.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678268547730-33eb2bb4-f8dc-4c6d-b871-d23954b8703f.jpeg#averageHue=%23ecebeb&clientId=uddebc9c7-8495-4&from=ui&id=ucf76f756&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=281996&status=done&style=none&taskId=u3dce2079-c124-4902-a3d3-396be616b47&title=)

![36.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678268547705-22484353-09bf-448b-bb99-9858a16a510a.jpeg#averageHue=%23ececec&clientId=uddebc9c7-8495-4&from=ui&id=u4dc1a841&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=227732&status=done&style=none&taskId=u4440164b-3057-4ea1-a2cf-82d8f08dd37&title=)

![37.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678268549720-310f0893-f3f6-400d-b461-1bd9c397a148.jpeg#averageHue=%23ecebeb&clientId=uddebc9c7-8495-4&from=ui&id=u4b946567&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=233755&status=done&style=none&taskId=u579de94d-3dfa-4d2f-b7e9-c1cbb83022b&title=)

![38.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678268550030-f34a120f-ad86-4bab-9642-a19b35cf1c5b.jpeg#averageHue=%23ebebeb&clientId=uddebc9c7-8495-4&from=ui&id=uf957a472&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=240937&status=done&style=none&taskId=u11a3820e-2553-4dcc-8634-16f1e6846cb&title=)

## Clustered Indexes

The table is stored in the sort order speciﬁed by the primary key, as either heap- or index-organized storage. Since some DBMSs always use a clustered index, they will automatically make a hidden row id primary key if a table doesn’t have an explicit one, but others cannot use them at all.

![39.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678268550307-43e2644a-237e-44e0-a123-8f8d8ef26d36.jpeg#averageHue=%23ececec&clientId=uddebc9c7-8495-4&from=ui&id=uf5b5f0e7&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=313853&status=done&style=none&taskId=u90d6a9e7-21bd-4615-a225-9cdc66e4d1b&title=)

![40.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678268550474-9ac27422-214a-4526-a29b-189a4e738715.jpeg#averageHue=%23e9e9e9&clientId=uddebc9c7-8495-4&from=ui&id=u2091bbb6&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=328998&status=done&style=none&taskId=u03ccbf37-cafb-4b32-a788-18aaf4ccec5&title=)

## Heap Clustering

Tuples are sorted in the heap’s pages using the order speciﬁed by a clustering index. DBMS can jump directly to the pages if clustering index’s attributes are used to access tuples.

![41.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678268550653-cbe482d7-85af-4b89-9569-c16d8f046cb8.jpeg#averageHue=%23e9e8e8&clientId=uddebc9c7-8495-4&from=ui&id=ue9f7aac5&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=337514&status=done&style=none&taskId=u60a34a7b-2811-4cec-8b85-ed95f7dd52e&title=)

## Index Scan Page Sorting

Since directly retrieving tuples from an unclustered index is inefﬁcient, the DBMS can ﬁrst ﬁgure out all the tuples that it needs and then sort them based on their page id.

![42.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678268552125-50daecc8-598d-4945-9f8d-adde8fa7dc7e.jpeg#averageHue=%23e5e4e4&clientId=uddebc9c7-8495-4&from=ui&id=uf9ef74a7&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=471644&status=done&style=none&taskId=u16850d51-79ce-453d-96a9-5c86f952c1b&title=)

# B+Tree Design Choices

![43.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678268551758-a798885b-3205-477a-9e21-30cb02eb0d60.jpeg#averageHue=%23eeeeee&clientId=uddebc9c7-8495-4&from=ui&id=uec16384d&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=258236&status=done&style=none&taskId=ubb137c68-b3a1-4ef4-a659-a6a7d6fdd1c&title=)

## Node Size

**Depending on the storage medium**, we may prefer larger or smaller node sizes. For example, nodes stored on hard drives are usually on the order of megabytes in size to reduce the number of seeks needed to ﬁnd data and amortize the expensive disk read over a large chunk of data, while in-memory databases may use page sizes as small as 512 bytes in order to ﬁt the entire page into the CPU cache as well as to decrease data fragmentation. This choice can also be dependent on the type of workload, as point queries would prefer as small a page as possible to reduce the amount of unnecessary extra info loaded, while a large sequential scan might prefer large pages to reduce the number of fetches it needs to do.

![44.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678268552322-d6d13cca-6ad5-4ba6-bb8c-324db050138f.jpeg#averageHue=%23eeeeee&clientId=uddebc9c7-8495-4&from=ui&id=ub9635f64&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=264527&status=done&style=none&taskId=u25c12354-6c24-4dc9-ab27-94e29aeb469&title=)

## Merge Threshold

While B+Trees have a rule about merging underﬂowed nodes after a delete, sometimes it may be beneﬁcial to temporarily violate the rule to reduce the number of deletion operation. For instance, eager merging could lead to thrashing, where a lot of successive delete and insert operations lead to constant splits and merges. It also allows for **batched merging** where multiple merge operations happen all at once, reducing the amount of time that expensive write latches have to be taken on the tree.

![45.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678268552632-c5858097-3a90-492f-89a7-7f3012486e05.jpeg#averageHue=%23ececec&clientId=uddebc9c7-8495-4&from=ui&id=uce6f0aec&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=289656&status=done&style=none&taskId=u53b11b5a-4993-4172-8e54-165257d1574&title=)

## Variable Length Keys

Currently we have only discussed B+Trees with ﬁxed length keys. However we may also want to support variable length keys, such as the case where a small subset of large keys lead to a lot of wasted space. There are several approaches to this:

1. **Pointers**
   Instead of storing the keys directly, we could just store a pointer to the key. Due to the inefﬁciency of having to chase a pointer for each key, the only place that uses this method in production is *embedded devices*, where its tiny registers and cache may beneﬁt from such space savings
2. **Variable Length Nodes
   **We could also still store the keys like normal and allow for variable length nodes. This is infeasible and largely not used due to the signiﬁcant memory management overhead of dealing with variable length nodes.
3. **Padding**
   Instead of varying the key size, we could set each key’s size to the size of the maximum key and pad out all the shorter keys. In most cases this is a massive waste of memory, so you don’t see this used by anyone either.
4. **Key Map/Indirection**
   The method that nearly everyone uses is replacing the keys with an index to the key-value pair in a separate dictionary. This offers signiﬁcant space savings and potentially shortcuts point queries (since the key-value pair the index points to is the exact same as the one pointed to by leaf nodes). Due to the small size of the dictionary index value, there is enough space to place a preﬁx of each key alongside the index, potentially allowing some index searching and leaf scanning to not even have to chase the pointer (if the preﬁx is at all different from the search key).

> **Pointer chasing** refers to a common sequence of instructions that involves a repeated series of irregular [memory access patterns](https://en.wikichip.org/w/index.php?title=memory_access_patterns&action=edit&redlink=1) that require the accessed data to determine the subsequent pointer address to be accessed, forming a serially-dependent chain of loads.

![46.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678268552874-492cd464-acb5-419e-ad38-236015cac736.jpeg#averageHue=%23eaeaea&clientId=uddebc9c7-8495-4&from=ui&id=u10b0c9b2&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=374763&status=done&style=none&taskId=u57666855-1530-4f41-bff1-f8f72743ac4&title=)

![Screen Shot 2023-03-09 at 2.18.55 PM.png](https://cdn.nlark.com/yuque/0/2023/png/22382307/1678342760432-6ee6b6f2-5327-4633-9b7f-9d53294c7de1.png#averageHue=%23f0efef&clientId=u8531362c-abde-4&from=ui&id=u04788e88&originHeight=1110&originWidth=1894&originalType=binary&ratio=2&rotation=0&showTitle=false&size=180529&status=done&style=none&taskId=u9a4e4222-489e-4eaa-b884-990267c0ff7&title=)

# Intra-Node Search

Once we reach a node, we still need to search within the node (either to ﬁnd the next node from an inner node, or to ﬁnd our key value in a leaf node). While this is relatively simple, there are still some tradeoffs to consider:

1. **Linear**
   The simplest solution is to just scan every key in the node until we ﬁnd our key. On the one hand, we don’t have to worry about sorting the keys, making insertions and deletes much quicker. On the other hand, this is relatively inefﬁcient and has a complexity of $O(n)$ per search. This can be vectorized using SIMD (or equivalent) instructions.
2. **Binary**
   A more efﬁcient solution for searching would be to keep each node sorted and use binary search to ﬁnd the key. This is as simple as jumping to the middle of a node and pivoting left or right depending on the comparison between the keys. Searches are much more efﬁcient this way, as this method only has the complexity of $O(ln(n))$ per search. However, insertions become more expensive as we must maintain the sort of each node.
3. **Interpolation**
   Finally, in some circumstances we may be able to utilize interpolation to ﬁnd the key. This method takes advantage of any metadata stored about the node (such as max element, min element, average, etc.) and uses it to generate an approximate location of the key. For example, if we are looking for 8 in a node and we know that 10 is the max key and $10 − (n + 1)$ is the smallest key (where n is the number of keys in each node), then we know to start searching 2 slots down from the max key, as the key one slot away from the max key must be 9 in this case. Despite being the fastest method we have given, this method is only seen in academic databases due to its limited applicability to keys with certain properties (like integers) and complexity.

![47.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678268553754-ae8081ff-60b7-4c83-8980-73afec3d198c.jpeg#averageHue=%23e8e7e7&clientId=uddebc9c7-8495-4&from=ui&id=u6f3f0004&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=389524&status=done&style=none&taskId=u214d44eb-97dd-4779-830b-3dde40c4162&title=)

# Optimizations

![48.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678268554312-abbdc093-1666-4814-a3e5-3a70c06ca4d3.jpeg#averageHue=%23efefef&clientId=uddebc9c7-8495-4&from=ui&id=u662b7b6f&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=212116&status=done&style=none&taskId=u5933d945-b997-4801-9927-baa45d129f3&title=)

## Preﬁx Compression

Most of the time when we have keys in the same node there will be some partial overlap of some preﬁx of each key (as **similar keys will end up right next to each other** in a sorted B+Tree). Instead of storing this preﬁx as part of each key multiple times, we can simply store the preﬁx once at the beginning of the node and then only include the unique sections of each key in each slot.

![49.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678268554613-45d4aa23-a320-42b6-af90-6611e068ace5.jpeg#averageHue=%23eaeaea&clientId=uddebc9c7-8495-4&from=ui&id=ude6c4702&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=317717&status=done&style=none&taskId=u6b0724fd-d24a-4172-b267-68968323b39&title=)

## Deduplication

In the case of an index which allows non-unique keys, we may end up with leaf nodes containing the same key over and over with different values attached. One optimization of this could be only writing the key once and then following it with all of its associated values.

![50.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678268554770-76485d4e-3fe2-4d18-92e4-78047fabb244.jpeg#averageHue=%23ebebeb&clientId=uddebc9c7-8495-4&from=ui&id=u57f31d8c&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=325274&status=done&style=none&taskId=u04a1cd22-3d66-46a4-9cb5-919e175e0d5&title=)

## Sufﬁx Truncation

For the most part the key entries in inner nodes are just used as signposts and not for their actual key value (as even if a key exists in the index we still have to search to the bottom to ensure that it hasn’t been deleted).
We can take advantage of this by only storing the **minimum preﬁx that is needed to correctly route** probes into the correct node.

![51.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678268555368-328d1ce0-c4bf-4ff1-97a9-764e889fa565.jpeg#averageHue=%23ebebeb&clientId=uddebc9c7-8495-4&from=ui&id=uaa6f1e2a&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=292052&status=done&style=none&taskId=ud23d447d-cdac-4ff4-b7b6-3c77d79cf09&title=)

![52.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678268556800-876936e2-1711-4bab-9ffb-f348789b92c7.jpeg#averageHue=%23ebebeb&clientId=uddebc9c7-8495-4&from=ui&id=u76f81ff9&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=282940&status=done&style=none&taskId=u9722caa9-abe7-489a-9afc-ba1acc0435a&title=)

## Pointer Swizzling

Because each node of a B+Tree is stored in a page from the buffer pool, each time we load a new page we need to fetch it from the buffer pool, requiring latching and lookups. To skip this step entirely, we could **store the actual raw pointers in place of the page IDs** (known as ”**swizzling**”), preventing a buffer pool fetch entirely. Rather than manually fetching the entire tree and placing the pointers manually, we can simply store the resulting pointer from a page lookup when traversing the index normally. Note that we must track which pointers are swizzled and deswizzle them back to page ids when the page they point to is unpinned and victimized.

![53.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678268557724-bfedcc2d-a6f3-4b37-b94f-9d7d454c23ea.jpeg#averageHue=%23e6e5e5&clientId=uddebc9c7-8495-4&from=ui&id=ua18799ec&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=412632&status=done&style=none&taskId=u5bbfcb8d-8f73-4bd4-971f-dab462989ee&title=)

![54.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678268558710-52bcf00e-f04c-4d46-8a07-1491e8e775a2.jpeg#averageHue=%23e5e5e5&clientId=uddebc9c7-8495-4&from=ui&id=uac6c40b2&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=434390&status=done&style=none&taskId=ub2d20d53-c519-4d9e-b0c5-e735d5a8444&title=)

![55.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678268558722-e903e6fb-2fdc-4117-bbb2-95da775efaa5.jpeg#averageHue=%23e6e5e5&clientId=uddebc9c7-8495-4&from=ui&id=u5b0b93c4&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=449095&status=done&style=none&taskId=u2ff0a1f5-2312-43eb-a3ec-a77406c6301&title=)

![56.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678268558843-052f3a2d-e640-4925-85f4-27c65400aca1.jpeg#averageHue=%23e6e5e5&clientId=uddebc9c7-8495-4&from=ui&id=u9f21c414&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=424227&status=done&style=none&taskId=ud2a7a7ba-5349-4ce6-a244-01e9f6aa996&title=)

## Bulk Insert

When a B+Tree is initially built, having to insert each key the usual way would lead to constant split operations. Since we already give leaves sibling pointers, initial insertion of data is much more efﬁcient if we construct a sorted linked list of leaf nodes and then easily build the index from the bottom up using the ﬁrst key from each leaf node. Note that depending on our context we may wish to pack the leaves as tightly as possible to save space or leave space in each leaf to allow for more inserts before a split is necessary.

![57.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678268559162-dd22e201-88df-4c7c-be24-cc076b80da33.jpeg#averageHue=%23ebebeb&clientId=uddebc9c7-8495-4&from=ui&id=u6a69bd7d&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=297869&status=done&style=none&taskId=uab10b6f3-c9a1-4a25-978f-00fa49f5ce6&title=)

![58.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678268559586-b902a752-225e-4d2e-817d-85429524ad7a.jpeg#averageHue=%23f1f1f1&clientId=uddebc9c7-8495-4&from=ui&id=uce46f3b3&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=153576&status=done&style=none&taskId=u210a8ca7-ad4d-42c6-a03c-bc0122c5307&title=)

![59.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678268560474-c7cf231d-7c44-4f2b-8ddc-37d93f85a8ca.jpeg#averageHue=%23f0f0f0&clientId=uddebc9c7-8495-4&from=ui&id=u850b9e61&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=180791&status=done&style=none&taskId=u164dc89b-9240-4e1d-a492-3386e96403a&title=)

![60.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678268560459-d0239b0d-1695-4019-99b3-1e3dbe9f58ff.jpeg#averageHue=%23f1f1f1&clientId=uddebc9c7-8495-4&from=ui&id=ue97f363c&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=148711&status=done&style=none&taskId=uf1e61546-bfa2-459e-b5ec-f614762f2de&title=)
