# Feedback deployment
The metadata feedback system is designed to associate metadata strings with the specimens stored at biobanks. A single sample could be associated with multiple strings. The most obvious use case is associating DOIs with specimens. The idea is that when a researcher starts work on a project involving specimens obtained from biobanks, they create a request ID for it and let the relevant biobanks know what the ID is. When the research is finished and the results have been published, the DOI of the publication is propagated to all relevant sites and associated with the samples via the request ID.

This project is based on the work of Adam Repasky for his masters thesis.

In this document, the installation of the components will be discussed, followed by a detailed description of the usage.

## Installation of components
The components of the metadata feedback system fall into two groups: those that are installed centrally (for the researcher: feedback hub) and those that are installed at the biobank sites in their Bridgeheads (for the biobank admin: feedback agent).

### Installation at biobanks
A prequisite for the use of metadata feedback at biobank sites is that they have installed a [Bridgehead](https://github.com/samply/bridgehead) and filled its Blaze store with data. More details about the process of setting up metadata feedback at a biobank can be found [here](https://github.com/samply/bridgehead/tree/metadata_fb?tab=readme-ov-file#metadata-feedback).

### Installation of central components
Full details of the installation of the central components can be found [here](https://github.com/samply/feedback-hub).

## Working with metada feedback
### Depositing data

Starting with the researcher, they should keep a table where they note the
queries that they perform with the Locator. This could be done by hand, but
a spreadsheet would be easier. The table sould have the following columns:

- Date and time (accuracy to the nearest minute would be good)
- Query (e.g. copy and paste from query bar of Locator)
- List of sites chosen for negotiation (empty means that no negotiation was done)
- Were specimens recieved as result of negotiation (Y/N)
- Were specimens used in research leading to publication (Y/N)
- Request ID (arbitrary, researcher can invent this)
- Publication ID (DOI, e.g. http://dx.doi.org/10.0000/0001)

Depending on the results of the various queries, many rows in the table will
be incomplete, i.e. not all columns will be filled. Only those rows where
every column is filled are suitable for the next steps.

Once the publication has been published, the researcher should
send an email to the BBMRI mailing list, asking for their metadata to be
included in the Bridgeheads. They can consult the table described above and
include date & time, query, list of sites and request ID in the email.

The BBMRI mailing list admin then forwards the email to the sites that provided
the data.

A site admin can query Blaze via the browser to find the relevant Measure Report:

https://localhost/bbmri-localdatamanagement/fhir/MeasureReport

It is quite possible that the time of the researcher's query is enough to
find the relevant measure report, but it may also be necessary to use the
researcher's query to find exactly the right measure report, depending on how
many researcher's queries were being run at the time.

The measure report ID can then be entered into the Feedback Agent's "Measure
Report ID" field and the generated "FHIR-query" can then be augmented with
the query paramaters supplied by the researcher to fetch the specimens.
Quite possibly it would be OK to select all of the specimens, before then
entering the request ID and submitting.

After all site admins have entered the data, the researcher uses the Feedback Hub
to send request ID and publication ID to Bridgeheads.

### Finding data

A typical use case would be "I have an ID for a publication. Can you tell me which
sites have samples for this publication?"

To realize this, a researcher would send the ID to the BBMRI support email. The
ID would then be distributed to all sites. At each site, the side admin would do
the following:

``` code
docker ps|grep feedback-agent-db
```

The first item printed out is the ID of the database container for the Feedback
Agent. This can be used to run queries against the database, e.g.:

``` code
docker exec -it a064b9a03769 psql -U compose-postgres
```

Within PSQL, the following commands can be carries out:

``` code
\c compose-postgres
\dt
select * from specimen_feedback where request_id='PublicationId';
\q
```

This sets the database and then runs the appropriate SQL to run a query with the
publication ID supplied by the researcher. Depending on the nature of the query,
the site admin can then send counts of specimens and patients to the mailing list
admin, who can then pass that information back to the researcher. In principle,
more detailed information could be provided, e.g. by using the specimen and patient
IDs to run queries against Blaze. However, for these detailed queries, some kind
of agreement between the researcher and the site would be necessary.

## TODOs
There are many ways in which metadata feedback could be improved:

- Integration into Negotiator, so that researchers and biobank site administrators do not need to use the BBMRI support email for every step. This would also mean that the researcher would no longer need to keep a separate list of searches, this functionality could be incorporated into the Negotiator.
- Basic auth for the feedback agent as a stopgap.
- Integration of the feedback agent into the Teiler.
- Optional automated addition of metadata at sites, without site admin intervention.
- Feedback agent: CORS currentls accepts all endpoints, this needs to be fixed.
- Feedback agent: Getting reference token requires direct communication with feedback hub. It would be better to do this via Beam.
- Feedbucj hub UI: VITE_BACKEND_URI is set in .env at build time. This needs to be settable at execution time, so that it can be passed as an environment variable to Docker.
