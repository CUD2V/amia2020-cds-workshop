# SAMPLE CODE USED IN THE TUTORIAL

# Editing Artifact with Tool

## Grade B Recommendation

#### Recommendation: 
<p>
Start low to moderate intensity lipid lowering therapy based on outcome of shared decision making between patient and provider
</p>

#### Rationale: 
<p>
The USPSTF found adequate evidence that use of low- to moderate-dose statins reduces the probability of CVD events (MI or ischemic stroke) and mortality by at least a moderate amount in adults aged 40 to 75 years who have 1 or more CVD risk factors (dyslipidemia, diabetes, hypertension, or smoking) and a calculated 10-year CVD event risk of 10% or greater.
</p>

## Grade C Recommendation

#### Recommendation: 
<p>
Discuss initiation of low to moderate intensity lowering therapy
</p>

#### Rationale: 
<p>
The USPSTF found adequate evidence that use of low- to moderate-dose statins reduces the probability of CVD events and mortality by at least a small amount in adults aged 40 to 75 years who have 1 or more CVD risk factors (dyslipidemia, diabetes, hypertension, or smoking) and a calculated 10-year CVD event risk of 7.5% to 10%.
</p>

# Editing Artifact Manually

## Add Inclusion Criterion

#### Add Value Set
```
valueset "Current Tobacco Smoker VS": 'https://cts.nlm.nih.gov/fhir/ValueSet/2.16.840.1.113883.3.600.2390'
```

#### Add Code
```
code "Tobacco smoking status code": '72166-2' from "LOINC" display 'Tobacco smoking status'
```

#### Define Is Smoker
```
define "Is Smoker":
  ConceptValue(
    MostRecent(
      Verified(
        ObservationLookBack(
          [Observation: "Tobacco smoking status code"], 
          6 years
        )
      )
    )
  ) in "Current Tobacco Smoker VS"
```

#### Additional criterion for group "One or More Risk Factors"
```
or "Is Smoker"
```

## Add Exclusion Criterion

#### Add Value Set
```
valueset "Myocardial Infarction VS": 'https://cts.nlm.nih.gov/fhir/ValueSet/2.16.840.1.113883.3.526.3.403' 
valueset "Acute Myocardial Infarction VS": 'https://cts.nlm.nih.gov/fhir/ValueSet/2.16.840.1.113883.3.464.1003.104.12.1001' 
valueset "Ischemic Vascular Disease VS": 'https://cts.nlm.nih.gov/fhir/ValueSet/2.16.840.1.113883.3.464.1003.104.12.1003'
```

#### Define CVD Condition Value Set
```
define "CVD_condition_valuesets":
  [Condition: "Myocardial Infarction VS"]
  union [Condition: "Acute Myocardial Infarction VS"]
  union [Condition: "Ischemic Vascular Disease VS"]
```

#### Define Has CVD
```
define "Has CVD":
  exists(
    Confirmed(
      "CVD_condition_valuesets"
    )
  )
```

#### Additional exclusion criteria
```
or "Has CVD"
```

#### Define RecommendationGrade
```
define "RecommendationGrade":
  case
    when "Grade B" then 'B'
    when "Grade C" then 'C'
    else null
  end
```

# CQL to ELM

#### Translate CQL
```
cql-to-elm --format=JSON --input ~/cql-results/src/data/R4/Statin-Use.cql
```

# Setup CQL Services

#### Stop cql-services
```
docker stop cql-services
```

#### CDS Hooks Template
```
{
  "id": "statin-use-r4",
  "hook": "patient-view",
  "title": "Statin Use for FHIR R4",
  "description": "Presents a United States Preventive Services Task Force (USPSTF) statin therapy recommendation for adults aged 40 to 75 years without a history of cardiovascular disease (CVD) who have 1 or more CVD risk factors (i.e., dyslipidemia, diabetes, hypertension, or smoking) and a calculated 10-year CVD event risk score of 7.5% or greater.",
  "_config": {
    "cards": [
       ** REPLACE WITH CARDS **
    ],
    "cql": {
      "library": {
        "id": "Statin-Use",
        "version": "1.0.0"
      }
    }
  }
}
```

#### CDS Hooks - Card 1
```
{
   "conditionExpression": "InPopulation",
   "card": {
      "summary": "Statin Use for the Primary Prevention of CVD in Adults",
      "indicator": "info",
      "detail": "${Recommendation}",
      "source": {
         "label": "CDS Connect: Statin Use for the Primary Prevention of CVD in Adults",
         "url": "https://cds.ahrq.gov/cdsconnect/artifact/statin-use-primary-prevention-cvd-adults"
      },
      "extension": {
         "grade": "${RecommendationGrade}",
         "rationale": "${Rationale}"
      }
   }
} 
```

#### CDS Hooks - Card 2
```
,{
   "conditionExpression": "Grade C",
   "card": {
      "summary": "Shared Decision Aid Tool for Statin Choice",
      "indicator": "info",
      "detail": "Discuss initiation of low to moderate intensity lipid lowering therapy",
      "links": [
         {
            "label": "Statin Choice Decision Aid",
            "url": "https://statindecisionaid.mayoclinic.org/",
            "type": "absolute"
         }
      ]
   }
}
```

### Start cql-services
```
docker start cql-services
```
