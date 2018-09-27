```@meta
CurrentModule = BioServices
```

# E-Utilities

E-Utilities provide a interface to Entrez databases at
[NCBI](https://www.ncbi.nlm.nih.gov/).  The APIs are defined in the
`BioServices.EUtils` module, which exports nine functions to access its databases:

| Function    | Description                                                                |
| :-------    | :----------                                                                |
| `einfo`     | Retrieve a list of databases or statistics for a database.                 |
| `esearch`   | Retrieve a list of UIDs matching a text query.                             |
| `epost`     | Upload or append a list of UIDs to the Entrez History server.              |
| `esummary`  | Retrieve document summaries for a list of UIDs.                            |
| `efetch`    | Retrieve formatted data records for a list of UIDs.                        |
| `elink`     | Retrieve UIDs linked to an input set of UIDs.                              |
| `egquery`   | Retrieve the number of available records in all databases by a text query. |
| `espell`    | Retrieve spelling suggestions.                                             |
| `ecitmatch` | Retrieve PubMed IDs that correspond to a set of input citation strings.    |

["The Nine E-utilities in
Brief"](https://www.ncbi.nlm.nih.gov/books/NBK25497/#_chapter2_The_Nine_Eutilities_in_Brief_)
summarizes all of the server-side programs corresponding to each function.

In this package, queries for databases are controlled by keyword parameters. For
example, some functions take `db` parameter to specify the target database.
Functions listed above take these parameters as keyword arguments and return
a `Response` object as follows:
```julia
julia> using BioServices.EUtils      # import the nine functions above

julia> res = einfo(db="pubmed")       # retrieve statistics of the PubMed database
HTTP.Messages.Response:
"""
HTTP/1.1 200 OK
Date: Thu, 27 Sep 2018 02:35:00 GMT
Server: Finatra
Strict-Transport-Security: max-age=31536000; includeSubDomains; preload
Content-Security-Policy: upgrade-insecure-requests
X-RateLimit-Remaining: 99
NCBI-PHID: F74BDE5F04CE9E9DDFEAD1C26D27933A.1.m_1
Cache-Control: private
NCBI-SID: 795BF1C3DA022296_2A99SID
X-RateLimit-Limit: 100
Access-Control-Allow-Origin: *
Content-Type: text/xml; charset=UTF-8
Set-Cookie: ncbi_sid=795BF1C3DA022296_2A99SID; domain=.nih.gov; path=/; expires=Fri, 27 Sep 2019 02:35:00 GMT
Vary: Accept-Encoding
X-UA-Compatible: IE=Edge
X-XSS-Protection: 1; mode=block
Transfer-Encoding: chunked

<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE eInfoResult PUBLIC "-//NLM//DTD einfo 20130322//EN" "https://eutils.ncbi.nlm.nih.gov/eutils/dtd/20130322/einfo.dtd">
<eInfoResult>
	<DbInfo>
	<DbName>pubmed</DbName>
	<MenuName>PubMed</MenuName>
	<Description>PubMed bibliographic record</Description>
	<DbBuild>Build180925-2212m.2</DbBuild>
	<Count>28891663</Count>
	<LastUpdate>2018/09/26 15:58</LastUpdate>
	<FieldList>
		<Field>
			<Name>ALL</Name>
			<FullName>All Fields</FullName>
			<Description>All terms from all searchable fields</Description>
			<TermCount>205729811</TermCount>
			<IsDate>N</IsDate>
			<IsNumerical>N</IsNumerical>
			<SingleToken>N</SingleToken>
			<Hierarchy>N</Hierarchy>
			<IsHidden>N</IsHidden>
		</Field>
		<Field>
			<Name>UID</Name>
			<FullName>UID</FullName>
			<Description>Unique number assigned to publication</Description>
			<TermCount>0</TermCount>
			<IsDate>N</IsDate>
			<IsNumerical>Y</IsNumerical>
			<SingleToken>Y</SingleToken>
			<Hierarchy>N</Hi
⋮
27368-byte body
"""

julia> write("pubmed.xml", res.body)  # save retrieved data into a file
27360

shell> head pubmed.xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE eInfoResult PUBLIC "-//NLM//DTD einfo 20130322//EN" "https://eutils.ncbi.nlm.nih.gov/eutils/dtd/20130322/einfo.dtd">
<eInfoResult>
	<DbInfo>
	<DbName>pubmed</DbName>
	<MenuName>PubMed</MenuName>
	<Description>PubMed bibliographic record</Description>
	<DbBuild>Build180925-2212m.2</DbBuild>
	<Count>28891663</Count>
	<LastUpdate>2018/09/26 15:58</LastUpdate>
```

Let's see a few more examples of parameters.  The `term` parameter specifies a
search string (e.g. `esearch(db="gene", term="tumor AND human[ORGN]")`).  The
`id` parameter specifies a UID (or accession number) or a list of UIDs (e.g.
`efetch(db="protein", id="NP_000537.3", rettype="fasta")`, `efetch(db="snp",
id=["rs55863639", "rs587780067"])`). The complete list of parameters can be
found at ["The E-utilities In-Depth: Parameters, Syntax and
More"](https://www.ncbi.nlm.nih.gov/books/NBK25499/).

When a request succeeds the response object has a `data` field containing
formatted data, which can be saved to a file as demonstrated above. However,
users are often interested in a part of the response data and may want to
extract some fields in it. In such a case,
[EzXML.jl](https://github.com/bicycle1885/EzXML.jl) is helpful because it offers
lots of tools to handle XML documents. The first thing you need to do is
converting the response data into an XML document by `parsexml`:
```julia
julia> res = efetch(db="nuccore", id="NM_001126.3", retmode="xml")
HTTP.Messages.Response:
"""
HTTP/1.1 200 OK
Date: Thu, 27 Sep 2018 02:37:57 GMT
Server: Finatra
Strict-Transport-Security: max-age=31536000; includeSubDomains; preload
Content-Security-Policy: upgrade-insecure-requests
Cache-Control: private
NCBI-PHID: 66264AA2CBBC7950185CA4CB710969F1.1.m_3
X-RateLimit-Remaining: 99
NCBI-SID: 08E5A6918EA30F95_A564SID
X-RateLimit-Limit: 100
Access-Control-Allow-Origin: *
Content-Type: text/xml
Content-Disposition: attachment; filename="sequence.gbx.xml"
Set-Cookie: ncbi_sid=08E5A6918EA30F95_A564SID; domain=.nih.gov; path=/; expires=Fri, 27 Sep 2019 02:37:57 GMT
Vary: Accept-Encoding
X-UA-Compatible: IE=Edge
X-XSS-Protection: 1; mode=block
Transfer-Encoding: chunked

<?xml version="1.0" encoding="UTF-8"  ?>
<!DOCTYPE GBSet PUBLIC "-//NCBI//NCBI GBSeq/EN" "https://www.ncbi.nlm.nih.gov/dtd/NCBI_GBSeq.dtd">
<GBSet>
  <GBSeq>

    <GBSeq_locus>NM_001126</GBSeq_locus>
    <GBSeq_length>2791</GBSeq_length>
    <GBSeq_strandedness>single</GBSeq_strandedness>
    <GBSeq_moltype>mRNA</GBSeq_moltype>
    <GBSeq_topology>linear</GBSeq_topology>
    <GBSeq_division>PRI</GBSeq_division>
    <GBSeq_update-date>25-JUN-2018</GBSeq_update-date>
    <GBSeq_create-date>01-APR-1999</GBSeq_create-date>
    <GBSeq_definition>Homo sapiens adenylosuccinate synthase (ADSS), mRNA</GBSeq_definition>
    <GBSeq_primary-accession>NM_001126</GBSeq_primary-accession>
    <GBSeq_accession-version>NM_001126.3</GBSeq_accession-version>
    <GBSeq_other-seqids>
      <GBSeqid>ref|NM_001126.3|</GBSeqid>
      <GBSeqid>gi|312836839</GBSeqid>
    </GBSeq_other-seqids>
    <GBSeq_keywords>
      <GBKeyword>RefSeq</GBKeyword>
    </GBSeq_keywords>
    <GBSeq_source>Homo sapiens (human)</
⋮
43391-byte body
"""

julia> using EzXML

julia> doc = parsexml(res.body)
EzXML.Document(EzXML.Node(<DOCUMENT_NODE@0x00007fdd4cc43770>))

```

After that, you can query fields you want using [XPath](https://en.wikipedia.org/wiki/XPath):
```julia
julia> seq = findfirst("/GBSet/GBSeq", doc)
EzXML.Node(<ELEMENT_NODE@0x00007fdd49f34b10>)

julia> nodecontent(findfirst("GBSeq_definition", seq))
"Homo sapiens adenylosuccinate synthase (ADSS), mRNA"

julia> nodecontent(findfirst(seq, "GBSeq_accession-version"))
"NM_001126.3"

julia> length(findall("//GBReference", seq))
10

julia> using Bio.Seq

julia> DNASequence(content(findfirst(seq, "GBSeq_sequence")))
2791nt DNA Sequence:
ACGGGAGTGGCGCGCCAGGCCGCGGAAGGGGCGTGGCCT…TGATTAAAAGAACCAAATATTTCTAGTATGAAAAAAAAA

```

Every function can take a context dictionary as its first argument to set
parameters into a query. Key-value pairs in a context are appended to a query in
addition to other parameters passed by keyword arguments.  The default context
is an empty dictionary that sets no parameters. This context dictionary is
especially useful when temporarily caching query UIDs into the Entrez History
server. A request to the Entrez system can be associated with cached data using
"WebEnv" and "query_key" parameters. In the following example, the search
results of `esearch` is saved in the Entrez History server (note
`usehistory=true`, which makes the server cache its search results) and then
their summaries are retrieved in the next call of `esummary`:
```julia
julia> context = Dict()  # create an empty context
Dict{Any,Any} with 0 entries

julia> res = esearch(context, db="pubmed", term="asthma[mesh] AND leukotrienes[mesh] AND 2009[pdat]", usehistory=true)
HTTP.Messages.Response:
"""
HTTP/1.1 200 OK
Date: Thu, 27 Sep 2018 02:42:30 GMT
Server: Finatra
Strict-Transport-Security: max-age=31536000; includeSubDomains; preload
Content-Security-Policy: upgrade-insecure-requests
X-RateLimit-Remaining: 99
NCBI-PHID: 8EB208709B8C2F742E7D63909B68BF8F.1.m_2
Cache-Control: private
NCBI-SID: DB29BA69C6192E09_B9A4SID
X-RateLimit-Limit: 100
Access-Control-Allow-Origin: *
Content-Type: text/xml; charset=UTF-8
Set-Cookie: ncbi_sid=DB29BA69C6192E09_B9A4SID; domain=.nih.gov; path=/; expires=Fri, 27 Sep 2019 02:42:31 GMT
Vary: Accept-Encoding
X-UA-Compatible: IE=Edge
X-XSS-Protection: 1; mode=block
Transfer-Encoding: chunked

<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE eSearchResult PUBLIC "-//NLM//DTD esearch 20060628//EN" "https://eutils.ncbi.nlm.nih.gov/eutils/dtd/20060628/esearch.dtd">
<eSearchResult><Count>56</Count><RetMax>20</RetMax><RetStart>0</RetStart><QueryKey>1</QueryKey><WebEnv>NCID_1_41597814_130.14.18.34_9002_1538016151_1639263172_0MetA0_S_MegaStore</WebEnv><IdList>
<Id>20113659</Id>
<Id>20074456</Id>
<Id>20046412</Id>
<Id>20021457</Id>
<Id>20008883</Id>
<Id>20008181</Id>
<Id>19912318</Id>
<Id>19897276</Id>
<Id>19895589</Id>
<Id>19894390</Id>
<Id>19852204</Id>
<Id>19839969</Id>
<Id>19811112</Id>
<Id>19757309</Id>
<Id>19749079</Id>
<Id>19739647</Id>
<Id>19706339</Id>
<Id>19665766</Id>
<Id>19648384</Id>
<Id>19647860</Id>
</IdList><TranslationSet><Translation>     <From>asthma[mesh]</From>     <To>"asthma"[MeSH Terms]</To>    </Translation><Translation>     <From>leukotrienes[mesh]</From>     <To>"leukotrienes"[MeSH Terms]</To>    </Translation></TranslationSet><TranslationStack>   <TermSe
⋮
1570-byte body
"""

julia> context  # the context dictionary has been updated
Dict{Any,Any} with 2 entries:
  :WebEnv    => "NCID_1_41597814_130.14.18.34_9002_1538016151_1639263172_0MetA0_S_MegaStore"
  :query_key => "1"

julia> res = esummary(context, db="pubmed")  # retrieve summaries in context
HTTP.Messages.Response:
"""
HTTP/1.1 200 OK
Date: Thu, 27 Sep 2018 02:43:23 GMT
Server: Finatra
Strict-Transport-Security: max-age=31536000; includeSubDomains; preload
Content-Security-Policy: upgrade-insecure-requests
X-RateLimit-Remaining: 99
NCBI-PHID: FE12C85A57CDC4C76E0B3434491AB527.1.m_1
Cache-Control: private
NCBI-SID: 6F1051FEDAC3C11A_D6B4SID
X-RateLimit-Limit: 100
Access-Control-Allow-Origin: *
Content-Type: text/xml; charset=UTF-8
Set-Cookie: ncbi_sid=6F1051FEDAC3C11A_D6B4SID; domain=.nih.gov; path=/; expires=Fri, 27 Sep 2019 02:43:23 GMT
Vary: Accept-Encoding
X-UA-Compatible: IE=Edge
X-XSS-Protection: 1; mode=block
Transfer-Encoding: chunked

<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE eSummaryResult PUBLIC "-//NLM//DTD esummary v1 20041029//EN" "https://eutils.ncbi.nlm.nih.gov/eutils/dtd/20041029/esummary-v1.dtd">
<eSummaryResult>
<DocSum>
	<Id>20113659</Id>
	<Item Name="PubDate" Type="Date">2009 Nov</Item>
	<Item Name="EPubDate" Type="Date"></Item>
	<Item Name="Source" Type="String">Zhongguo Dang Dai Er Ke Za Zhi</Item>
	<Item Name="AuthorList" Type="List">
		<Item Name="Author" Type="String">He MJ</Item>
		<Item Name="Author" Type="String">Chen Q</Item>
		<Item Name="Author" Type="String">Liu JM</Item>
	</Item>
	<Item Name="LastAuthor" Type="String">Liu JM</Item>
	<Item Name="Title" Type="String">[Urinary leukotrience E(4) level in children with asthma].</Item>
	<Item Name="Volume" Type="String">11</Item>
	<Item Name="Issue" Type="String">11</Item>
	<Item Name="Pages" Type="String">909-12</Item>
	<Item Name="LangList" Type="List">
		<Item Name="Lang" Type="String">Chinese</Item>
	</Item>
	<Item Name="NlmUniqueID" T
⋮
136322-byte body
"""

julia> write("asthma_leukotrienes_2009.xml", res.body)  # save data into a file
136322

shell> head asthma_leukotrienes_2009.xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE eSummaryResult PUBLIC "-//NLM//DTD esummary v1 20041029//EN" "https://eutils.ncbi.nlm.nih.gov/eutils/dtd/20041029/esummary-v1.dtd">
<eSummaryResult>
<DocSum>
        <Id>20113659</Id>
        <Item Name="PubDate" Type="Date">2009 Nov</Item>
        <Item Name="EPubDate" Type="Date"></Item>
        <Item Name="Source" Type="String">Zhongguo Dang Dai Er Ke Za Zhi</Item>
        <Item Name="AuthorList" Type="List">
                <Item Name="Author" Type="String">He MJ</Item>

```
