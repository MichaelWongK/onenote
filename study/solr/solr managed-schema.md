```
<?xml version="1.0" encoding="UTF-8" ?>
<!--
版本信息和说明
-->
<!--  
这是Solr模式文件。这个文件应该命名为“schema”。xml”,应该在conf目录下的solr home
(例如。/ solr / conf / schema.xml在默认情况下)或者定位到Solr webapp的类加载器可以找到它的位置。
这个示例模式是用户推荐的起点。它应该保持正确和简洁，可用的开箱即用。有关如何自定义此文件的更多信息，请参见(在slor7.5中是managed-schema的文件)
http://wiki.apache.org/solr/SchemaXml
性能注意:此模式包含许多可选特性，不应该包含用于基准测试。提高性能是可以的
 性能说明:可以如下来提高性能。
  - 设置 stored="false" 对那些只需要搜索，无需返回的字段.
  - 设置 indexed="false" 对于那些只用于返回无需进行搜索的字段.
  - 删除所有不需要 copyfiled字段的声明
  - 为了最好的索引大小与索引性能，设置所有一般的文本字段index=false,使用copyfile将他们copy到一个字段上，然后使用它进行搜索。
  - 运行jvm服务器模式，并使用较高的日志级别，避免记录每一个请求。
-->

<schema name="default-config" version="1.6">
  <!-- 该配置名称与版本说明.
    -->
<!-- 字段的有效属性:
name：属性的名称，这里有个特殊的属性“_version_”是必须添加的。
type：字段的数据结构类型，所用到的类型需要在fieldType中设置。
default：默认值。
indexed：是否创建索引
stored：是否存储原始数据（如果不需要存储相应字段值，尽量设为false）
docValues：表示此域是否需要添加一个 docValues 域，这对 facet 查询， group 分组，排序， function 查询有好处，尽管这个属性不是必须的，但他能加快索引数据加载，对 NRT 近实时搜索比较友好，且更节省内存，但它也有一些限制，比如当前docValues 域只支持 strField,UUIDField,Trie*Field 等域，且要求域的域值是单值不能是多值域
solrMissingFirst/solrMissingLast：查询结果排序的过程中，如果发现这个字段的值不存在，则排在前面/后面，忽略排序的条件
multValued：是否有多个值，比如说一个用户的所有好友id。（对可能存在多值的字段尽量设置为true，避免建索引时抛出错误）
omitNorms：此属性若设置为 true ，即表示将忽略域值的长度标准化，忽略在索引过程中对当前域的权重设置，且会节省内存。只有全文本域或者你需要在索引创建过程中设置域的权重时才需要把这个值设为 false, 对于基本数据类型且不分词的域如intFeild,longField,Stre, 否则默认就是 false.
required：添加文档时，该字段必须存在，类似mysql的not null
termVectors: 设置为 true 即表示需要为该 field 存储项向量信息，当你需要MoreLikeThis 功能时，则需要将此属性值设为 true ，这样会带来一些性能提升。
termPositions: 是否存储 Term 的起始位置信息，这会增大索引的体积，但高亮功能需要依赖此项设置，否则无法高亮
termOffsets: 表示是否存储索引的位置偏移量，高亮功能需要此项配置，当你使用SpanQuery 时，此项配置会影响匹配的结果集
-->
 <!--字段名称应该包含字母数字或下划线字符，不以一个数字开始。这是目前没有严格执行，但其他字段名称将不会有来自所有组件的第一类支持和背部的兼容性没有保证。领导和的名字下划线（如_version_）保留。-->

  <!--
在这data_driven_schema_configs configset，下面三个字段是必须的：
id、_version_，和_text_。所有其他字段都是可以删除修改的，并根据需要手动添加
在xml。
请注意，许多动态字段也被定义-您可以使用它们来指定一个
字段的类型通过字段命名约定-见下文。
警告：本_text_catch所字段将会显著地提高索引的大小。
如果你不需要，考虑删除它和相应的copyfield指令。-->
   <!-- 常规字段-->
    <field name="id" type="string" indexed="true" stored="true" required="true" multiValued="false" />
    <!-- 默认情况下，长类型启用docValues，所以我们不需要索引version字段  -->
	    <!--预留字段 -->
    <field name="_version_" type="plong" indexed="false" stored="false"/>
    <field name="_root_" type="string" indexed="true" stored="false" docValues="false" />
    <field name="_text_" type="text_general" indexed="true" stored="false" multiValued="true"/>

    <!-- 如果客户端不知道可以搜索哪些字段，可以启用此功能。它在默认情况下不启用因为把所有东西都索引两次是非常消耗资源的。 -->

	   <!--复制字段-->
    <!-- <copyField source="*" dest="_text_"/> -->

    <!--
最大的区别是name的属性上可以进行通配，比如说name="*_i"，那么只要是后面带i的字段都是符合的。这样就不怕一些字段无法匹配无法写入 限制:name属性中的类全局模式只能在开头或结尾有一个“*”。
-->
   <!--动态字段-->
    <dynamicField name="*_i"  type="pint"    indexed="true"  stored="true"/>
    <dynamicField name="*_is" type="pints"    indexed="true"  stored="true"/>
    <dynamicField name="*_s"  type="string"  indexed="true"  stored="true" />
    <dynamicField name="*_ss" type="strings"  indexed="true"  stored="true"/>
    <dynamicField name="*_l"  type="plong"   indexed="true"  stored="true"/>
    <dynamicField name="*_ls" type="plongs"   indexed="true"  stored="true"/>
    <dynamicField name="*_t" type="text_general" indexed="true" stored="true" multiValued="false"/>
    <dynamicField name="*_txt" type="text_general" indexed="true" stored="true"/>
    <dynamicField name="*_b"  type="boolean" indexed="true" stored="true"/>
    <dynamicField name="*_bs" type="booleans" indexed="true" stored="true"/>
    <dynamicField name="*_f"  type="pfloat"  indexed="true"  stored="true"/>
    <dynamicField name="*_fs" type="pfloats"  indexed="true"  stored="true"/>
    <dynamicField name="*_d"  type="pdouble" indexed="true"  stored="true"/>
    <dynamicField name="*_ds" type="pdoubles" indexed="true"  stored="true"/>
    <dynamicField name="random_*" type="random"/>

    <!--用于数据驱动模式的类型，用于为每个文本字段添加字符串副本-->
    <dynamicField name="*_str" type="strings" stored="false" docValues="true" indexed="false" useDocValuesAsStored="false"/>

    <dynamicField name="*_dt"  type="pdate"    indexed="true"  stored="true"/>
    <dynamicField name="*_dts" type="pdate"    indexed="true"  stored="true" multiValued="true"/>
    <dynamicField name="*_p"  type="location" indexed="true" stored="true"/>
    <dynamicField name="*_srpt"  type="location_rpt" indexed="true" stored="true"/>
    
    <!-- 加载动态字段-->
    <dynamicField name="*_dpf" type="delimited_payloads_float" indexed="true"  stored="true"/>
    <dynamicField name="*_dpi" type="delimited_payloads_int" indexed="true"  stored="true"/>
    <dynamicField name="*_dps" type="delimited_payloads_string" indexed="true"  stored="true"/>

    <dynamicField name="attr_*" type="text_general" indexed="true" stored="true" multiValued="true"/>

    <!-- 用于确定和强制文档唯一性的字段。除非这个字段被标记为required="false"，否则它将是一个必填字段
    -->
    <uniqueKey>id</uniqueKey>

    <!--
StrField: 这是一个不分词的字符串域，它支持 docValues 域，但当为其添加了docValues 域，则要求只能是单值域且该域必须存在或者该域有默认值
BoolField ： boolean 域，对应 true/false
TrieIntField, TrieFloatField, TrieLongField, TrieDoubleField 这几个都是默认的数字域， precisionStep 属性一般用于数字范围查询， precisionStep 值越小，则索引时该域的域值分出的 token 个数越多，会增大硬盘上索引的体积，但它会加快数字范围检索的响应速度， positionIncrementGap 属性表示如果当前域是多值域时，多个值之间的间距，单值域，设置此项无意义。
TrieDateField ：显然这是一个日期域类型，不过遗憾的是它支持 1995-12-31T23:59:59Z 这种格式的日期，比较坑爹，为此我自定义了一个 TrieCNDateField 域类型，用于支持国人比较喜欢的 yyyy-MM-dd HH:mm:ss 格式的日期。源码请参见我的上一篇博客。
BinaryField ：经过 base64 编码的字符串域类型，即你需要把 binary 数据进行base64 编码才能被 solr 进行索引。
RandomSortField ：随机排序域类型，当你需要实现伪随机排序时，请使用此域类型。
TextField ：是用的最多的一种域类型，它需要进行分词，所以它一般需要配置分词器。至于具体它如何配置 IK 分词器，这里就不展开了-->
<!-- 字段类型 -->
    <fieldType name="string" class="solr.StrField" sortMissingLast="true" docValues="true" />
    <fieldType name="strings" class="solr.StrField" sortMissingLast="true" multiValued="true" docValues="true" />

    <!-- boolean type: "true" or "false" -->
    <fieldType name="boolean" class="solr.BoolField" sortMissingLast="true"/>
    <fieldType name="booleans" class="solr.BoolField" sortMissingLast="true" multiValued="true"/>

    <!--
     使用kd树索引值的数值字段类型。
点字段不支持FieldCache，因此如果需要排序、faceting和函数等，它们必须具有docValues="true"。
    -->
	
    <fieldType name="pint" class="solr.IntPointField" docValues="true"/>
    <fieldType name="pfloat" class="solr.FloatPointField" docValues="true"/>
    <fieldType name="plong" class="solr.LongPointField" docValues="true"/>
    <fieldType name="pdouble" class="solr.DoublePointField" docValues="true"/>
    
    <fieldType name="pints" class="solr.IntPointField" docValues="true" multiValued="true"/>
    <fieldType name="pfloats" class="solr.FloatPointField" docValues="true" multiValued="true"/>
    <fieldType name="plongs" class="solr.LongPointField" docValues="true" multiValued="true"/>
    <fieldType name="pdoubles" class="solr.DoublePointField" docValues="true" multiValued="true"/>
    <fieldType name="random" class="solr.RandomSortField" indexed="true"/>


    <!-- The format for this date field is of the form 1995-12-31T23:59:59Z, and
         is a more restricted form of the canonical representation of dateTime
         http://www.w3.org/TR/xmlschema-2/#dateTime    
         The trailing "Z" designates UTC time and is mandatory.
         Optional fractional seconds are allowed: 1995-12-31T23:59:59.999Z
         All other components are mandatory.

         Expressions can also be used to denote calculations that should be
         performed relative to "NOW" to determine the value, ie...

               NOW/HOUR
                  ... Round to the start of the current hour
               NOW-1DAY
                  ... Exactly 1 day prior to now
               NOW/DAY+6MONTHS+3DAYS
                  ... 6 months and 3 days in the future from the start of
                      the current day
                      
      -->
    <!-- KD-tree versions of date fields -->
    <fieldType name="pdate" class="solr.DatePointField" docValues="true"/>
    <fieldType name="pdates" class="solr.DatePointField" docValues="true" multiValued="true"/>
    
    <!--Binary data type. The data should be sent/retrieved in as Base64 encoded Strings -->
    <fieldType name="binary" class="solr.BinaryField"/>

    <!-- solr.TextField allows the specification of custom text analyzers
         specified as a tokenizer and a list of token filters. Different
         analyzers may be specified for indexing and querying.

         The optional positionIncrementGap puts space between multiple fields of
         this type on the same document, with the purpose of preventing false phrase
         matching across fields.

         For more info on customizing your analyzer chain, please see
         http://lucene.apache.org/solr/guide/understanding-analyzers-tokenizers-and-filters.html#understanding-analyzers-tokenizers-and-filters
     -->

    <!-- One can also specify an existing Analyzer class that has a
         default constructor via the class attribute on the analyzer element.
         Example:
    <fieldType name="text_greek" class="solr.TextField">
      <analyzer class="org.apache.lucene.analysis.el.GreekAnalyzer"/>
    </fieldType>
    -->

    <!-- A text field that only splits on whitespace for exact matching of words -->
    <dynamicField name="*_ws" type="text_ws"  indexed="true"  stored="true"/>
    <fieldType name="text_ws" class="solr.TextField" positionIncrementGap="100">
      <analyzer>
        <tokenizer class="solr.WhitespaceTokenizerFactory"/>
      </analyzer>
    </fieldType>

    <!-- A general text field that has reasonable, generic
         cross-language defaults: it tokenizes with StandardTokenizer,
	       removes stop words from case-insensitive "stopwords.txt"
	       (empty by default), and down cases.  At query time only, it
	       also applies synonyms.
	  -->
    <fieldType name="text_general" class="solr.TextField" positionIncrementGap="100" multiValued="true">
      <analyzer type="index">
        <tokenizer class="solr.StandardTokenizerFactory"/>
        <filter class="solr.StopFilterFactory" ignoreCase="true" words="stopwords.txt" />
        <!-- in this example, we will only use synonyms at query time
        <filter class="solr.SynonymGraphFilterFactory" synonyms="index_synonyms.txt" ignoreCase="true" expand="false"/>
        <filter class="solr.FlattenGraphFilterFactory"/>
        -->
        <filter class="solr.LowerCaseFilterFactory"/>
      </analyzer>
      <analyzer type="query">
        <tokenizer class="solr.StandardTokenizerFactory"/>
        <filter class="solr.StopFilterFactory" ignoreCase="true" words="stopwords.txt" />
        <filter class="solr.SynonymGraphFilterFactory" synonyms="synonyms.txt" ignoreCase="true" expand="true"/>
        <filter class="solr.LowerCaseFilterFactory"/>
      </analyzer>
    </fieldType>

    
    <!-- SortableTextField generaly functions exactly like TextField,
         except that it supports, and by default uses, docValues for sorting (or faceting)
         on the first 1024 characters of the original field values (which is configurable).
         
         This makes it a bit more useful then TextField in many situations, but the trade-off
         is that it takes up more space on disk; which is why it's not used in place of TextField
         for every fieldType in this _default schema.
	  -->
    <dynamicField name="*_t_sort" type="text_gen_sort" indexed="true" stored="true" multiValued="false"/>
    <dynamicField name="*_txt_sort" type="text_gen_sort" indexed="true" stored="true"/>
    <fieldType name="text_gen_sort" class="solr.SortableTextField" positionIncrementGap="100" multiValued="true">
      <analyzer type="index">
        <tokenizer class="solr.StandardTokenizerFactory"/>
        <filter class="solr.StopFilterFactory" ignoreCase="true" words="stopwords.txt" />
        <filter class="solr.LowerCaseFilterFactory"/>
      </analyzer>
      <analyzer type="query">
        <tokenizer class="solr.StandardTokenizerFactory"/>
        <filter class="solr.StopFilterFactory" ignoreCase="true" words="stopwords.txt" />
        <filter class="solr.SynonymGraphFilterFactory" synonyms="synonyms.txt" ignoreCase="true" expand="true"/>
        <filter class="solr.LowerCaseFilterFactory"/>
      </analyzer>
    </fieldType>

    <!-- A text field with defaults appropriate for English: it tokenizes with StandardTokenizer,
         removes English stop words (lang/stopwords_en.txt), down cases, protects words from protwords.txt, and
         finally applies Porter's stemming.  The query time analyzer also applies synonyms from synonyms.txt. -->
    <dynamicField name="*_txt_en" type="text_en"  indexed="true"  stored="true"/>
    <fieldType name="text_en" class="solr.TextField" positionIncrementGap="100">
      <analyzer type="index">
        <tokenizer class="solr.StandardTokenizerFactory"/>
        <!-- in this example, we will only use synonyms at query time
        <filter class="solr.SynonymGraphFilterFactory" synonyms="index_synonyms.txt" ignoreCase="true" expand="false"/>
        <filter class="solr.FlattenGraphFilterFactory"/>
        -->
        <!-- Case insensitive stop word removal.
        -->
        <filter class="solr.StopFilterFactory"
                ignoreCase="true"
                words="lang/stopwords_en.txt"
            />
        <filter class="solr.LowerCaseFilterFactory"/>
        <filter class="solr.EnglishPossessiveFilterFactory"/>
        <filter class="solr.KeywordMarkerFilterFactory" protected="protwords.txt"/>
        <!-- Optionally you may want to use this less aggressive stemmer instead of PorterStemFilterFactory:
        <filter class="solr.EnglishMinimalStemFilterFactory"/>
	      -->
        <filter class="solr.PorterStemFilterFactory"/>
      </analyzer>
      <analyzer type="query">
        <tokenizer class="solr.StandardTokenizerFactory"/>
        <filter class="solr.SynonymGraphFilterFactory" synonyms="synonyms.txt" ignoreCase="true" expand="true"/>
        <filter class="solr.StopFilterFactory"
                ignoreCase="true"
                words="lang/stopwords_en.txt"
        />
        <filter class="solr.LowerCaseFilterFactory"/>
        <filter class="solr.EnglishPossessiveFilterFactory"/>
        <filter class="solr.KeywordMarkerFilterFactory" protected="protwords.txt"/>
        <!-- Optionally you may want to use this less aggressive stemmer instead of PorterStemFilterFactory:
        <filter class="solr.EnglishMinimalStemFilterFactory"/>
	      -->
        <filter class="solr.PorterStemFilterFactory"/>
      </analyzer>
    </fieldType>

    <!-- A text field with defaults appropriate for English, plus
         aggressive word-splitting and autophrase features enabled.
         This field is just like text_en, except it adds
         WordDelimiterGraphFilter to enable splitting and matching of
         words on case-change, alpha numeric boundaries, and
         non-alphanumeric chars.  This means certain compound word
         cases will work, for example query "wi fi" will match
         document "WiFi" or "wi-fi".
    -->
    <dynamicField name="*_txt_en_split" type="text_en_splitting"  indexed="true"  stored="true"/>
    <fieldType name="text_en_splitting" class="solr.TextField" positionIncrementGap="100" autoGeneratePhraseQueries="true">
      <analyzer type="index">
        <tokenizer class="solr.WhitespaceTokenizerFactory"/>
        <!-- in this example, we will only use synonyms at query time
        <filter class="solr.SynonymGraphFilterFactory" synonyms="index_synonyms.txt" ignoreCase="true" expand="false"/>
        -->
        <!-- Case insensitive stop word removal.
        -->
        <filter class="solr.StopFilterFactory"
                ignoreCase="true"
                words="lang/stopwords_en.txt"
        />
        <filter class="solr.WordDelimiterGraphFilterFactory" generateWordParts="1" generateNumberParts="1" catenateWords="1" catenateNumbers="1" catenateAll="0" splitOnCaseChange="1"/>
        <filter class="solr.LowerCaseFilterFactory"/>
        <filter class="solr.KeywordMarkerFilterFactory" protected="protwords.txt"/>
        <filter class="solr.PorterStemFilterFactory"/>
        <filter class="solr.FlattenGraphFilterFactory" />
      </analyzer>
      <analyzer type="query">
        <tokenizer class="solr.WhitespaceTokenizerFactory"/>
        <filter class="solr.SynonymGraphFilterFactory" synonyms="synonyms.txt" ignoreCase="true" expand="true"/>
        <filter class="solr.StopFilterFactory"
                ignoreCase="true"
                words="lang/stopwords_en.txt"
        />
        <filter class="solr.WordDelimiterGraphFilterFactory" generateWordParts="1" generateNumberParts="1" catenateWords="0" catenateNumbers="0" catenateAll="0" splitOnCaseChange="1"/>
        <filter class="solr.LowerCaseFilterFactory"/>
        <filter class="solr.KeywordMarkerFilterFactory" protected="protwords.txt"/>
        <filter class="solr.PorterStemFilterFactory"/>
      </analyzer>
    </fieldType>

    <!-- Less flexible matching, but less false matches.  Probably not ideal for product names,
         but may be good for SKUs.  Can insert dashes in the wrong place and still match. -->
    <dynamicField name="*_txt_en_split_tight" type="text_en_splitting_tight"  indexed="true"  stored="true"/>
    <fieldType name="text_en_splitting_tight" class="solr.TextField" positionIncrementGap="100" autoGeneratePhraseQueries="true">
      <analyzer type="index">
        <tokenizer class="solr.WhitespaceTokenizerFactory"/>
        <filter class="solr.SynonymGraphFilterFactory" synonyms="synonyms.txt" ignoreCase="true" expand="false"/>
        <filter class="solr.StopFilterFactory" ignoreCase="true" words="lang/stopwords_en.txt"/>
        <filter class="solr.WordDelimiterGraphFilterFactory" generateWordParts="0" generateNumberParts="0" catenateWords="1" catenateNumbers="1" catenateAll="0"/>
        <filter class="solr.LowerCaseFilterFactory"/>
        <filter class="solr.KeywordMarkerFilterFactory" protected="protwords.txt"/>
        <filter class="solr.EnglishMinimalStemFilterFactory"/>
        <!-- this filter can remove any duplicate tokens that appear at the same position - sometimes
             possible with WordDelimiterGraphFilter in conjuncton with stemming. -->
        <filter class="solr.RemoveDuplicatesTokenFilterFactory"/>
        <filter class="solr.FlattenGraphFilterFactory" />
      </analyzer>
      <analyzer type="query">
        <tokenizer class="solr.WhitespaceTokenizerFactory"/>
        <filter class="solr.SynonymGraphFilterFactory" synonyms="synonyms.txt" ignoreCase="true" expand="false"/>
        <filter class="solr.StopFilterFactory" ignoreCase="true" words="lang/stopwords_en.txt"/>
        <filter class="solr.WordDelimiterGraphFilterFactory" generateWordParts="0" generateNumberParts="0" catenateWords="1" catenateNumbers="1" catenateAll="0"/>
        <filter class="solr.LowerCaseFilterFactory"/>
        <filter class="solr.KeywordMarkerFilterFactory" protected="protwords.txt"/>
        <filter class="solr.EnglishMinimalStemFilterFactory"/>
        <!-- this filter can remove any duplicate tokens that appear at the same position - sometimes
             possible with WordDelimiterGraphFilter in conjuncton with stemming. -->
        <filter class="solr.RemoveDuplicatesTokenFilterFactory"/>
      </analyzer>
    </fieldType>

    <!-- Just like text_general except it reverses the characters of
	       each token, to enable more efficient leading wildcard queries.
    -->
    <dynamicField name="*_txt_rev" type="text_general_rev"  indexed="true"  stored="true"/>
    <fieldType name="text_general_rev" class="solr.TextField" positionIncrementGap="100">
      <analyzer type="index">
        <tokenizer class="solr.StandardTokenizerFactory"/>
        <filter class="solr.StopFilterFactory" ignoreCase="true" words="stopwords.txt" />
        <filter class="solr.LowerCaseFilterFactory"/>
        <filter class="solr.ReversedWildcardFilterFactory" withOriginal="true"
                maxPosAsterisk="3" maxPosQuestion="2" maxFractionAsterisk="0.33"/>
      </analyzer>
      <analyzer type="query">
        <tokenizer class="solr.StandardTokenizerFactory"/>
        <filter class="solr.SynonymGraphFilterFactory" synonyms="synonyms.txt" ignoreCase="true" expand="true"/>
        <filter class="solr.StopFilterFactory" ignoreCase="true" words="stopwords.txt" />
        <filter class="solr.LowerCaseFilterFactory"/>
      </analyzer>
    </fieldType>

    <dynamicField name="*_phon_en" type="phonetic_en"  indexed="true"  stored="true"/>
    <fieldType name="phonetic_en" stored="false" indexed="true" class="solr.TextField" >
      <analyzer>
        <tokenizer class="solr.StandardTokenizerFactory"/>
        <filter class="solr.DoubleMetaphoneFilterFactory" inject="false"/>
      </analyzer>
    </fieldType>

    <!-- lowercases the entire field value, keeping it as a single token.  -->
    <dynamicField name="*_s_lower" type="lowercase"  indexed="true"  stored="true"/>
    <fieldType name="lowercase" class="solr.TextField" positionIncrementGap="100">
      <analyzer>
        <tokenizer class="solr.KeywordTokenizerFactory"/>
        <filter class="solr.LowerCaseFilterFactory" />
      </analyzer>
    </fieldType>

    <!-- 
      Example of using PathHierarchyTokenizerFactory at index time, so
      queries for paths match documents at that path, or in descendent paths
    -->
    <dynamicField name="*_descendent_path" type="descendent_path"  indexed="true"  stored="true"/>
    <fieldType name="descendent_path" class="solr.TextField">
      <analyzer type="index">
        <tokenizer class="solr.PathHierarchyTokenizerFactory" delimiter="/" />
      </analyzer>
      <analyzer type="query">
        <tokenizer class="solr.KeywordTokenizerFactory" />
      </analyzer>
    </fieldType>

    <!--
      Example of using PathHierarchyTokenizerFactory at query time, so
      queries for paths match documents at that path, or in ancestor paths
    -->
    <dynamicField name="*_ancestor_path" type="ancestor_path"  indexed="true"  stored="true"/>
    <fieldType name="ancestor_path" class="solr.TextField">
      <analyzer type="index">
        <tokenizer class="solr.KeywordTokenizerFactory" />
      </analyzer>
      <analyzer type="query">
        <tokenizer class="solr.PathHierarchyTokenizerFactory" delimiter="/" />
      </analyzer>
    </fieldType>

    <!-- This point type indexes the coordinates as separate fields (subFields)
      If subFieldType is defined, it references a type, and a dynamic field
      definition is created matching *___<typename>.  Alternately, if 
      subFieldSuffix is defined, that is used to create the subFields.
      Example: if subFieldType="double", then the coordinates would be
        indexed in fields myloc_0___double,myloc_1___double.
      Example: if subFieldSuffix="_d" then the coordinates would be indexed
        in fields myloc_0_d,myloc_1_d
      The subFields are an implementation detail of the fieldType, and end
      users normally should not need to know about them.
     -->
    <dynamicField name="*_point" type="point"  indexed="true"  stored="true"/>
    <fieldType name="point" class="solr.PointType" dimension="2" subFieldSuffix="_d"/>

    <!-- A specialized field for geospatial search filters and distance sorting. -->
    <fieldType name="location" class="solr.LatLonPointSpatialField" docValues="true"/>

    <!-- A geospatial field type that supports multiValued and polygon shapes.
      For more information about this and other spatial fields see:
      http://lucene.apache.org/solr/guide/spatial-search.html
    -->
    <fieldType name="location_rpt" class="solr.SpatialRecursivePrefixTreeFieldType"
               geo="true" distErrPct="0.025" maxDistErr="0.001" distanceUnits="kilometers" />

    <!-- Payloaded field types -->
    <fieldType name="delimited_payloads_float" stored="false" indexed="true" class="solr.TextField">
      <analyzer>
        <tokenizer class="solr.WhitespaceTokenizerFactory"/>
        <filter class="solr.DelimitedPayloadTokenFilterFactory" encoder="float"/>
      </analyzer>
    </fieldType>
    <fieldType name="delimited_payloads_int" stored="false" indexed="true" class="solr.TextField">
      <analyzer>
        <tokenizer class="solr.WhitespaceTokenizerFactory"/>
        <filter class="solr.DelimitedPayloadTokenFilterFactory" encoder="integer"/>
      </analyzer>
    </fieldType>
    <fieldType name="delimited_payloads_string" stored="false" indexed="true" class="solr.TextField">
      <analyzer>
        <tokenizer class="solr.WhitespaceTokenizerFactory"/>
        <filter class="solr.DelimitedPayloadTokenFilterFactory" encoder="identity"/>
      </analyzer>
    </fieldType>

    <!-- some examples for different languages (generally ordered by ISO code) -->

    <!-- Arabic -->
    <dynamicField name="*_txt_ar" type="text_ar"  indexed="true"  stored="true"/>
    <fieldType name="text_ar" class="solr.TextField" positionIncrementGap="100">
      <analyzer> 
        <tokenizer class="solr.StandardTokenizerFactory"/>
        <!-- for any non-arabic -->
        <filter class="solr.LowerCaseFilterFactory"/>
        <filter class="solr.StopFilterFactory" ignoreCase="true" words="lang/stopwords_ar.txt" />
        <!-- normalizes ﻯ to ﻱ, etc -->
        <filter class="solr.ArabicNormalizationFilterFactory"/>
        <filter class="solr.ArabicStemFilterFactory"/>
      </analyzer>
    </fieldType>

    <!-- Bulgarian -->
    <dynamicField name="*_txt_bg" type="text_bg"  indexed="true"  stored="true"/>
    <fieldType name="text_bg" class="solr.TextField" positionIncrementGap="100">
      <analyzer> 
        <tokenizer class="solr.StandardTokenizerFactory"/> 
        <filter class="solr.LowerCaseFilterFactory"/>
        <filter class="solr.StopFilterFactory" ignoreCase="true" words="lang/stopwords_bg.txt" /> 
        <filter class="solr.BulgarianStemFilterFactory"/>       
      </analyzer>
    </fieldType>
    
    <!-- Catalan -->
    <dynamicField name="*_txt_ca" type="text_ca"  indexed="true"  stored="true"/>
    <fieldType name="text_ca" class="solr.TextField" positionIncrementGap="100">
      <analyzer> 
        <tokenizer class="solr.StandardTokenizerFactory"/>
        <!-- removes l', etc -->
        <filter class="solr.ElisionFilterFactory" ignoreCase="true" articles="lang/contractions_ca.txt"/>
        <filter class="solr.LowerCaseFilterFactory"/>
        <filter class="solr.StopFilterFactory" ignoreCase="true" words="lang/stopwords_ca.txt" />
        <filter class="solr.SnowballPorterFilterFactory" language="Catalan"/>       
      </analyzer>
    </fieldType>
    
    <!-- CJK bigram (see text_ja for a Japanese configuration using morphological analysis) -->
    <dynamicField name="*_txt_cjk" type="text_cjk"  indexed="true"  stored="true"/>
    <fieldType name="text_cjk" class="solr.TextField" positionIncrementGap="100">
      <analyzer>
        <tokenizer class="solr.StandardTokenizerFactory"/>
        <!-- normalize width before bigram, as e.g. half-width dakuten combine  -->
        <filter class="solr.CJKWidthFilterFactory"/>
        <!-- for any non-CJK -->
        <filter class="solr.LowerCaseFilterFactory"/>
        <filter class="solr.CJKBigramFilterFactory"/>
      </analyzer>
    </fieldType>

    <!-- Czech -->
    <dynamicField name="*_txt_cz" type="text_cz"  indexed="true"  stored="true"/>
    <fieldType name="text_cz" class="solr.TextField" positionIncrementGap="100">
      <analyzer> 
        <tokenizer class="solr.StandardTokenizerFactory"/>
        <filter class="solr.LowerCaseFilterFactory"/>
        <filter class="solr.StopFilterFactory" ignoreCase="true" words="lang/stopwords_cz.txt" />
        <filter class="solr.CzechStemFilterFactory"/>       
      </analyzer>
    </fieldType>
    
    <!-- Danish -->
    <dynamicField name="*_txt_da" type="text_da"  indexed="true"  stored="true"/>
    <fieldType name="text_da" class="solr.TextField" positionIncrementGap="100">
      <analyzer> 
        <tokenizer class="solr.StandardTokenizerFactory"/>
        <filter class="solr.LowerCaseFilterFactory"/>
        <filter class="solr.StopFilterFactory" ignoreCase="true" words="lang/stopwords_da.txt" format="snowball" />
        <filter class="solr.SnowballPorterFilterFactory" language="Danish"/>       
      </analyzer>
    </fieldType>
    
    <!-- German -->
    <dynamicField name="*_txt_de" type="text_de"  indexed="true"  stored="true"/>
    <fieldType name="text_de" class="solr.TextField" positionIncrementGap="100">
      <analyzer> 
        <tokenizer class="solr.StandardTokenizerFactory"/>
        <filter class="solr.LowerCaseFilterFactory"/>
        <filter class="solr.StopFilterFactory" ignoreCase="true" words="lang/stopwords_de.txt" format="snowball" />
        <filter class="solr.GermanNormalizationFilterFactory"/>
        <filter class="solr.GermanLightStemFilterFactory"/>
        <!-- less aggressive: <filter class="solr.GermanMinimalStemFilterFactory"/> -->
        <!-- more aggressive: <filter class="solr.SnowballPorterFilterFactory" language="German2"/> -->
      </analyzer>
    </fieldType>
    
    <!-- Greek -->
    <dynamicField name="*_txt_el" type="text_el"  indexed="true"  stored="true"/>
    <fieldType name="text_el" class="solr.TextField" positionIncrementGap="100">
      <analyzer> 
        <tokenizer class="solr.StandardTokenizerFactory"/>
        <!-- greek specific lowercase for sigma -->
        <filter class="solr.GreekLowerCaseFilterFactory"/>
        <filter class="solr.StopFilterFactory" ignoreCase="false" words="lang/stopwords_el.txt" />
        <filter class="solr.GreekStemFilterFactory"/>
      </analyzer>
    </fieldType>
    
    <!-- Spanish -->
    <dynamicField name="*_txt_es" type="text_es"  indexed="true"  stored="true"/>
    <fieldType name="text_es" class="solr.TextField" positionIncrementGap="100">
      <analyzer> 
        <tokenizer class="solr.StandardTokenizerFactory"/>
        <filter class="solr.LowerCaseFilterFactory"/>
        <filter class="solr.StopFilterFactory" ignoreCase="true" words="lang/stopwords_es.txt" format="snowball" />
        <filter class="solr.SpanishLightStemFilterFactory"/>
        <!-- more aggressive: <filter class="solr.SnowballPorterFilterFactory" language="Spanish"/> -->
      </analyzer>
    </fieldType>
    
    <!-- Basque -->
    <dynamicField name="*_txt_eu" type="text_eu"  indexed="true"  stored="true"/>
    <fieldType name="text_eu" class="solr.TextField" positionIncrementGap="100">
      <analyzer> 
        <tokenizer class="solr.StandardTokenizerFactory"/>
        <filter class="solr.LowerCaseFilterFactory"/>
        <filter class="solr.StopFilterFactory" ignoreCase="true" words="lang/stopwords_eu.txt" />
        <filter class="solr.SnowballPorterFilterFactory" language="Basque"/>
      </analyzer>
    </fieldType>
    
    <!-- Persian -->
    <dynamicField name="*_txt_fa" type="text_fa"  indexed="true"  stored="true"/>
    <fieldType name="text_fa" class="solr.TextField" positionIncrementGap="100">
      <analyzer>
        <!-- for ZWNJ -->
        <charFilter class="solr.PersianCharFilterFactory"/>
        <tokenizer class="solr.StandardTokenizerFactory"/>
        <filter class="solr.LowerCaseFilterFactory"/>
        <filter class="solr.ArabicNormalizationFilterFactory"/>
        <filter class="solr.PersianNormalizationFilterFactory"/>
        <filter class="solr.StopFilterFactory" ignoreCase="true" words="lang/stopwords_fa.txt" />
      </analyzer>
    </fieldType>
    
    <!-- Finnish -->
    <dynamicField name="*_txt_fi" type="text_fi"  indexed="true"  stored="true"/>
    <fieldType name="text_fi" class="solr.TextField" positionIncrementGap="100">
      <analyzer> 
        <tokenizer class="solr.StandardTokenizerFactory"/>
        <filter class="solr.LowerCaseFilterFactory"/>
        <filter class="solr.StopFilterFactory" ignoreCase="true" words="lang/stopwords_fi.txt" format="snowball" />
        <filter class="solr.SnowballPorterFilterFactory" language="Finnish"/>
        <!-- less aggressive: <filter class="solr.FinnishLightStemFilterFactory"/> -->
      </analyzer>
    </fieldType>
    
    <!-- French -->
    <dynamicField name="*_txt_fr" type="text_fr"  indexed="true"  stored="true"/>
    <fieldType name="text_fr" class="solr.TextField" positionIncrementGap="100">
      <analyzer> 
        <tokenizer class="solr.StandardTokenizerFactory"/>
        <!-- removes l', etc -->
        <filter class="solr.ElisionFilterFactory" ignoreCase="true" articles="lang/contractions_fr.txt"/>
        <filter class="solr.LowerCaseFilterFactory"/>
        <filter class="solr.StopFilterFactory" ignoreCase="true" words="lang/stopwords_fr.txt" format="snowball" />
        <filter class="solr.FrenchLightStemFilterFactory"/>
        <!-- less aggressive: <filter class="solr.FrenchMinimalStemFilterFactory"/> -->
        <!-- more aggressive: <filter class="solr.SnowballPorterFilterFactory" language="French"/> -->
      </analyzer>
    </fieldType>
    
    <!-- Irish -->
    <dynamicField name="*_txt_ga" type="text_ga"  indexed="true"  stored="true"/>
    <fieldType name="text_ga" class="solr.TextField" positionIncrementGap="100">
      <analyzer> 
        <tokenizer class="solr.StandardTokenizerFactory"/>
        <!-- removes d', etc -->
        <filter class="solr.ElisionFilterFactory" ignoreCase="true" articles="lang/contractions_ga.txt"/>
        <!-- removes n-, etc. position increments is intentionally false! -->
        <filter class="solr.StopFilterFactory" ignoreCase="true" words="lang/hyphenations_ga.txt"/>
        <filter class="solr.IrishLowerCaseFilterFactory"/>
        <filter class="solr.StopFilterFactory" ignoreCase="true" words="lang/stopwords_ga.txt"/>
        <filter class="solr.SnowballPorterFilterFactory" language="Irish"/>
      </analyzer>
    </fieldType>
    
    <!-- Galician -->
    <dynamicField name="*_txt_gl" type="text_gl"  indexed="true"  stored="true"/>
    <fieldType name="text_gl" class="solr.TextField" positionIncrementGap="100">
      <analyzer> 
        <tokenizer class="solr.StandardTokenizerFactory"/>
        <filter class="solr.LowerCaseFilterFactory"/>
        <filter class="solr.StopFilterFactory" ignoreCase="true" words="lang/stopwords_gl.txt" />
        <filter class="solr.GalicianStemFilterFactory"/>
        <!-- less aggressive: <filter class="solr.GalicianMinimalStemFilterFactory"/> -->
      </analyzer>
    </fieldType>
    
    <!-- Hindi -->
    <dynamicField name="*_txt_hi" type="text_hi"  indexed="true"  stored="true"/>
    <fieldType name="text_hi" class="solr.TextField" positionIncrementGap="100">
      <analyzer> 
        <tokenizer class="solr.StandardTokenizerFactory"/>
        <filter class="solr.LowerCaseFilterFactory"/>
        <!-- normalizes unicode representation -->
        <filter class="solr.IndicNormalizationFilterFactory"/>
        <!-- normalizes variation in spelling -->
        <filter class="solr.HindiNormalizationFilterFactory"/>
        <filter class="solr.StopFilterFactory" ignoreCase="true" words="lang/stopwords_hi.txt" />
        <filter class="solr.HindiStemFilterFactory"/>
      </analyzer>
    </fieldType>
    
    <!-- Hungarian -->
    <dynamicField name="*_txt_hu" type="text_hu"  indexed="true"  stored="true"/>
    <fieldType name="text_hu" class="solr.TextField" positionIncrementGap="100">
      <analyzer> 
        <tokenizer class="solr.StandardTokenizerFactory"/>
        <filter class="solr.LowerCaseFilterFactory"/>
        <filter class="solr.StopFilterFactory" ignoreCase="true" words="lang/stopwords_hu.txt" format="snowball" />
        <filter class="solr.SnowballPorterFilterFactory" language="Hungarian"/>
        <!-- less aggressive: <filter class="solr.HungarianLightStemFilterFactory"/> -->   
      </analyzer>
    </fieldType>
    
    <!-- Armenian -->
    <dynamicField name="*_txt_hy" type="text_hy"  indexed="true"  stored="true"/>
    <fieldType name="text_hy" class="solr.TextField" positionIncrementGap="100">
      <analyzer> 
        <tokenizer class="solr.StandardTokenizerFactory"/>
        <filter class="solr.LowerCaseFilterFactory"/>
        <filter class="solr.StopFilterFactory" ignoreCase="true" words="lang/stopwords_hy.txt" />
        <filter class="solr.SnowballPorterFilterFactory" language="Armenian"/>
      </analyzer>
    </fieldType>
    
    <!-- Indonesian -->
    <dynamicField name="*_txt_id" type="text_id"  indexed="true"  stored="true"/>
    <fieldType name="text_id" class="solr.TextField" positionIncrementGap="100">
      <analyzer> 
        <tokenizer class="solr.StandardTokenizerFactory"/>
        <filter class="solr.LowerCaseFilterFactory"/>
        <filter class="solr.StopFilterFactory" ignoreCase="true" words="lang/stopwords_id.txt" />
        <!-- for a less aggressive approach (only inflectional suffixes), set stemDerivational to false -->
        <filter class="solr.IndonesianStemFilterFactory" stemDerivational="true"/>
      </analyzer>
    </fieldType>
    
    <!-- Italian -->
  <dynamicField name="*_txt_it" type="text_it"  indexed="true"  stored="true"/>
  <fieldType name="text_it" class="solr.TextField" positionIncrementGap="100">
      <analyzer> 
        <tokenizer class="solr.StandardTokenizerFactory"/>
        <!-- removes l', etc -->
        <filter class="solr.ElisionFilterFactory" ignoreCase="true" articles="lang/contractions_it.txt"/>
        <filter class="solr.LowerCaseFilterFactory"/>
        <filter class="solr.StopFilterFactory" ignoreCase="true" words="lang/stopwords_it.txt" format="snowball" />
        <filter class="solr.ItalianLightStemFilterFactory"/>
        <!-- more aggressive: <filter class="solr.SnowballPorterFilterFactory" language="Italian"/> -->
      </analyzer>
    </fieldType>
    
    <!-- Japanese using morphological analysis (see text_cjk for a configuration using bigramming)

         NOTE: If you want to optimize search for precision, use default operator AND in your request
         handler config (q.op) Use OR if you would like to optimize for recall (default).
    -->
    <dynamicField name="*_txt_ja" type="text_ja"  indexed="true"  stored="true"/>
    <fieldType name="text_ja" class="solr.TextField" positionIncrementGap="100" autoGeneratePhraseQueries="false">
      <analyzer>
        <!-- Kuromoji Japanese morphological analyzer/tokenizer (JapaneseTokenizer)

           Kuromoji has a search mode (default) that does segmentation useful for search.  A heuristic
           is used to segment compounds into its parts and the compound itself is kept as synonym.

           Valid values for attribute mode are:
              normal: regular segmentation
              search: segmentation useful for search with synonyms compounds (default)
            extended: same as search mode, but unigrams unknown words (experimental)

           For some applications it might be good to use search mode for indexing and normal mode for
           queries to reduce recall and prevent parts of compounds from being matched and highlighted.
           Use <analyzer type="index"> and <analyzer type="query"> for this and mode normal in query.

           Kuromoji also has a convenient user dictionary feature that allows overriding the statistical
           model with your own entries for segmentation, part-of-speech tags and readings without a need
           to specify weights.  Notice that user dictionaries have not been subject to extensive testing.

           User dictionary attributes are:
                     userDictionary: user dictionary filename
             userDictionaryEncoding: user dictionary encoding (default is UTF-8)

           See lang/userdict_ja.txt for a sample user dictionary file.

           Punctuation characters are discarded by default.  Use discardPunctuation="false" to keep them.
        -->
        <tokenizer class="solr.JapaneseTokenizerFactory" mode="search"/>
        <!--<tokenizer class="solr.JapaneseTokenizerFactory" mode="search" userDictionary="lang/userdict_ja.txt"/>-->
        <!-- Reduces inflected verbs and adjectives to their base/dictionary forms (辞書形) -->
        <filter class="solr.JapaneseBaseFormFilterFactory"/>
        <!-- Removes tokens with certain part-of-speech tags -->
        <filter class="solr.JapanesePartOfSpeechStopFilterFactory" tags="lang/stoptags_ja.txt" />
        <!-- Normalizes full-width romaji to half-width and half-width kana to full-width (Unicode NFKC subset) -->
        <filter class="solr.CJKWidthFilterFactory"/>
        <!-- Removes common tokens typically not useful for search, but have a negative effect on ranking -->
        <filter class="solr.StopFilterFactory" ignoreCase="true" words="lang/stopwords_ja.txt" />
        <!-- Normalizes common katakana spelling variations by removing any last long sound character (U+30FC) -->
        <filter class="solr.JapaneseKatakanaStemFilterFactory" minimumLength="4"/>
        <!-- Lower-cases romaji characters -->
        <filter class="solr.LowerCaseFilterFactory"/>
      </analyzer>
    </fieldType>
    
    <!-- Korean morphological analysis -->
    <dynamicField name="*_txt_ko" type="text_ko"  indexed="true"  stored="true"/>
    <fieldType name="text_ko" class="solr.TextField" positionIncrementGap="100">
      <analyzer>
        <!-- Nori Korean morphological analyzer/tokenizer (KoreanTokenizer)
          The Korean (nori) analyzer integrates Lucene nori analysis module into Solr.
          It uses the mecab-ko-dic dictionary to perform morphological analysis of Korean texts.

          This dictionary was built with MeCab, it defines a format for the features adapted
          for the Korean language.
          
          Nori also has a convenient user dictionary feature that allows overriding the statistical
          model with your own entries for segmentation, part-of-speech tags and readings without a need
          to specify weights. Notice that user dictionaries have not been subject to extensive testing.

          The tokenizer supports multiple schema attributes:
            * userDictionary: User dictionary path.
            * userDictionaryEncoding: User dictionary encoding.
            * decompoundMode: Decompound mode. Either 'none', 'discard', 'mixed'. Default is 'discard'.
            * outputUnknownUnigrams: If true outputs unigrams for unknown words.
        -->
        <tokenizer class="solr.KoreanTokenizerFactory" decompoundMode="discard" outputUnknownUnigrams="false"/>
        <!-- Removes some part of speech stuff like EOMI (Pos.E), you can add a parameter 'tags',
          listing the tags to remove. By default it removes: 
          E, IC, J, MAG, MAJ, MM, SP, SSC, SSO, SC, SE, XPN, XSA, XSN, XSV, UNA, NA, VSV
          This is basically an equivalent to stemming.
        -->
        <filter class="solr.KoreanPartOfSpeechStopFilterFactory" />
        <!-- Replaces term text with the Hangul transcription of Hanja characters, if applicable: -->
        <filter class="solr.KoreanReadingFormFilterFactory" />
        <filter class="solr.LowerCaseFilterFactory" />
      </analyzer>
    </fieldType>

    <!-- Latvian -->
    <dynamicField name="*_txt_lv" type="text_lv"  indexed="true"  stored="true"/>
    <fieldType name="text_lv" class="solr.TextField" positionIncrementGap="100">
      <analyzer> 
        <tokenizer class="solr.StandardTokenizerFactory"/>
        <filter class="solr.LowerCaseFilterFactory"/>
        <filter class="solr.StopFilterFactory" ignoreCase="true" words="lang/stopwords_lv.txt" />
        <filter class="solr.LatvianStemFilterFactory"/>
      </analyzer>
    </fieldType>
    
    <!-- Dutch -->
    <dynamicField name="*_txt_nl" type="text_nl"  indexed="true"  stored="true"/>
    <fieldType name="text_nl" class="solr.TextField" positionIncrementGap="100">
      <analyzer> 
        <tokenizer class="solr.StandardTokenizerFactory"/>
        <filter class="solr.LowerCaseFilterFactory"/>
        <filter class="solr.StopFilterFactory" ignoreCase="true" words="lang/stopwords_nl.txt" format="snowball" />
        <filter class="solr.StemmerOverrideFilterFactory" dictionary="lang/stemdict_nl.txt" ignoreCase="false"/>
        <filter class="solr.SnowballPorterFilterFactory" language="Dutch"/>
      </analyzer>
    </fieldType>
    
    <!-- Norwegian -->
    <dynamicField name="*_txt_no" type="text_no"  indexed="true"  stored="true"/>
    <fieldType name="text_no" class="solr.TextField" positionIncrementGap="100">
      <analyzer> 
        <tokenizer class="solr.StandardTokenizerFactory"/>
        <filter class="solr.LowerCaseFilterFactory"/>
        <filter class="solr.StopFilterFactory" ignoreCase="true" words="lang/stopwords_no.txt" format="snowball" />
        <filter class="solr.SnowballPorterFilterFactory" language="Norwegian"/>
        <!-- less aggressive: <filter class="solr.NorwegianLightStemFilterFactory"/> -->
        <!-- singular/plural: <filter class="solr.NorwegianMinimalStemFilterFactory"/> -->
      </analyzer>
    </fieldType>
    
    <!-- Portuguese -->
  <dynamicField name="*_txt_pt" type="text_pt"  indexed="true"  stored="true"/>
  <fieldType name="text_pt" class="solr.TextField" positionIncrementGap="100">
      <analyzer> 
        <tokenizer class="solr.StandardTokenizerFactory"/>
        <filter class="solr.LowerCaseFilterFactory"/>
        <filter class="solr.StopFilterFactory" ignoreCase="true" words="lang/stopwords_pt.txt" format="snowball" />
        <filter class="solr.PortugueseLightStemFilterFactory"/>
        <!-- less aggressive: <filter class="solr.PortugueseMinimalStemFilterFactory"/> -->
        <!-- more aggressive: <filter class="solr.SnowballPorterFilterFactory" language="Portuguese"/> -->
        <!-- most aggressive: <filter class="solr.PortugueseStemFilterFactory"/> -->
      </analyzer>
    </fieldType>
    
    <!-- Romanian -->
    <dynamicField name="*_txt_ro" type="text_ro"  indexed="true"  stored="true"/>
    <fieldType name="text_ro" class="solr.TextField" positionIncrementGap="100">
      <analyzer> 
        <tokenizer class="solr.StandardTokenizerFactory"/>
        <filter class="solr.LowerCaseFilterFactory"/>
        <filter class="solr.StopFilterFactory" ignoreCase="true" words="lang/stopwords_ro.txt" />
        <filter class="solr.SnowballPorterFilterFactory" language="Romanian"/>
      </analyzer>
    </fieldType>
    
    <!-- Russian -->
    <dynamicField name="*_txt_ru" type="text_ru"  indexed="true"  stored="true"/>
    <fieldType name="text_ru" class="solr.TextField" positionIncrementGap="100">
      <analyzer> 
        <tokenizer class="solr.StandardTokenizerFactory"/>
        <filter class="solr.LowerCaseFilterFactory"/>
        <filter class="solr.StopFilterFactory" ignoreCase="true" words="lang/stopwords_ru.txt" format="snowball" />
        <filter class="solr.SnowballPorterFilterFactory" language="Russian"/>
        <!-- less aggressive: <filter class="solr.RussianLightStemFilterFactory"/> -->
      </analyzer>
    </fieldType>
    
    <!-- Swedish -->
    <dynamicField name="*_txt_sv" type="text_sv"  indexed="true"  stored="true"/>
    <fieldType name="text_sv" class="solr.TextField" positionIncrementGap="100">
      <analyzer> 
        <tokenizer class="solr.StandardTokenizerFactory"/>
        <filter class="solr.LowerCaseFilterFactory"/>
        <filter class="solr.StopFilterFactory" ignoreCase="true" words="lang/stopwords_sv.txt" format="snowball" />
        <filter class="solr.SnowballPorterFilterFactory" language="Swedish"/>
        <!-- less aggressive: <filter class="solr.SwedishLightStemFilterFactory"/> -->
      </analyzer>
    </fieldType>
    
    <!-- Thai -->
    <dynamicField name="*_txt_th" type="text_th"  indexed="true"  stored="true"/>
    <fieldType name="text_th" class="solr.TextField" positionIncrementGap="100">
      <analyzer>
        <tokenizer class="solr.ThaiTokenizerFactory"/>
        <filter class="solr.LowerCaseFilterFactory"/>
        <filter class="solr.StopFilterFactory" ignoreCase="true" words="lang/stopwords_th.txt" />
      </analyzer>
    </fieldType>
    
    <!-- Turkish -->
    <dynamicField name="*_txt_tr" type="text_tr"  indexed="true"  stored="true"/>
    <fieldType name="text_tr" class="solr.TextField" positionIncrementGap="100">
      <analyzer> 
        <tokenizer class="solr.StandardTokenizerFactory"/>
        <filter class="solr.TurkishLowerCaseFilterFactory"/>
        <filter class="solr.StopFilterFactory" ignoreCase="false" words="lang/stopwords_tr.txt" />
        <filter class="solr.SnowballPorterFilterFactory" language="Turkish"/>
      </analyzer>
    </fieldType>

    <!-- Similarity is the scoring routine for each document vs. a query.
       A custom Similarity or SimilarityFactory may be specified here, but 
       the default is fine for most applications.  
       For more info: http://lucene.apache.org/solr/guide/other-schema-elements.html#OtherSchemaElements-Similarity
    -->
    <!--
     <similarity class="com.example.solr.CustomSimilarityFactory">
       <str name="paramkey">param value</str>
     </similarity>
    -->

</schema>
```