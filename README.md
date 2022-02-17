# BiocCuratedCCDG_ANVIL
Manage concepts related to PFB import from Gen3 to AnVIL; example of CCDG

# AnVIL Workspace Description

## Concept

Given a PFB "export to terra" from Gen3, produce a DataFrame instance
that can be used as a 'sample metadata' resource for molecular data
lodged in the workspace.

Here are the fields of "sample" data from CCDG:

```
entity:sample_id	pfb:created_datetime	pfb:project_id	pfb:sample_provider	pfb:specimen_id	pfb:state	pfb:subject	pfb:submitter_id	pfb:updated_datetime
```

The function given below will take sample, subject, and sequencing tables and put them together with sample_id as the primary key.

## colData construction
```
build_colData = function() {
 require(AnVIL)
 require(dplyr)
 simplify_names = function(.data) {
   names(.data) = gsub("^pfb:", "", names(.data))
   .data
 }
 samp = avtable(table="sample")
 subj = avtable(table="subject")
 seqtab = avtable(table="sequencing")
#alltypes = unique(samp$`pfb:tissue_type`)
#tissframes = lapply(na.omit(alltypes), function(x) samp |> filter(`pfb:tissue_type` == x))
#names(tissframes) = na.omit(alltypes)
#library(MultiAssayExperiment)
# by stages
#t2 = lapply(tissframes, function(x) mutate(x, subject_id=`pfb:subject`))
 t3 = left_join(mutate(samp, subject_id=`pfb:subject`), subj, by="subject_id")
 sqsq = mutate(seqtab, sample_id=`pfb:sample`)
 inner_join(sqsq, t3, by="sample_id") |> simplify_names()
}
```

The result of running the above function in this workspace has fields:

```
> names(ccdg_full_coldata)
 [1] "sequencing_id"              "total_reads"                "file_format"                "project_id"                
 [5] "estimated_library_size"     "data_format"                "sequencing_assay"           "mean_coverage_per_base"    
 [9] "date_data_generation"       "data_type"                  "file_size"                  "md5sum"                    
[13] "experimental_strategy"      "file_name"                  "submitter_id"               "object_id"                 
[17] "file_state"                 "fragment_length_mean"       "file_type"                  "duplication_rate_of_mapped"
[21] "data_category"              "state"                      "reference_genome_build"     "created_datetime"          
[25] "ga4gh_drs_uri"              "file_md5sum"                "updated_datetime"           "sample"                    
[29] "sample_id"                  "project_id.x"               "submitter_id.x"             "sample_provider"           
[33] "state.x"                    "created_datetime.x"         "specimen_id"                "updated_datetime.x"        
[37] "subject"                    "subject_id"                 "sex"                        "project_id.y"              
[41] "participant_id"             "submitter_id.y"             "state.y"                    "dbgap_submission"          
[45] "created_datetime.y"         "dbgap_study_id"             "updated_datetime.y"         "project"          
```
