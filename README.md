# GuideR
User-guided separate-and-conquer rule learning in classification, regression, and survival settings

## Usage
UPDATE: GuideR is now the part of [RuleKit](https://github.com/adaa-polsl/RuleKit) - our comprehensive suite for rule-based learning. We suggest using RuleKit for analyses as we constatly improve its functionality and keep it up to date.

GuideR is distributed as a standalone JAR package (see Release tab for download). To run the analysis, execute
```
java -jar GuideR experiments.xml
```
where *experiments.xml* is an XML file with a description of experimental setting. It describes parameter sets and datasets to be examined: 
```
</experiment>
	<parameter_sets>
		<parameter_set name="paramset_1">...</parameter_set>
    	<parameter_set name="paramset_2">...</parameter_set>
    	...
  	</parameter_sets>

  	<datasets>
    	<dataset name="dataset_1">...</dataset>
    	<dataset name="dataset_2">...</dataset>
    	...
  	</datasets>
</experiment>
```
### Parameter set description

As each algorithm parameter has its default value, only selected parameters may specified by the user. In automatic mode, following parameters apply:

```
<parameter_set name="paramset_1">
  	<param name="min_rule_covered">...</param>
  	<param name="induction_measure">...</param>
  	<param name="pruning_measure">...</param>
	<param name="voting_measure">...</param>
</parameter_set>
```    
where:
* `min_rule_covered` - minimum number of previously uncovered examples a new rule has to cover,
* `induction_measure` - rule quality measure used during growing; one of the following: *Accuracy*, *C2*, *Correlation*, 		*Lift*,	*LogicalSufficiency*,	*Precision*, *RSS*,	*GeoRSS*, *SBayesian*, *BinaryEntropy*,
* `pruning_measure` - rule quality measure used during pruning; one of the aforementioned measures,
* `voting_measure` - rule quality measure used for voting; one of the aforementioned measures.

The measure parameters apply only for classification and regression problems - in survival datasets log-rank statistics is always used.

Expert knowledge is also specified through parameters:
```
<parameter_set name="paramset_1">
  	<param name="min_rule_covered">...</param>
  	<param name="induction_measure">...</param>
  	<param name="pruning_measure">...</param>
	<param name="voting_measure">...</param>
  	<param name="use_expert">true</param>
  	<param name="extend_using_preferred">...</param>
  	<param name="extend_using_automatic">...</param>
  	<param name="induce_using_preferred">...</param>
  	<param name="induce_using_automatic">...</param>
  	<param name="preferred_conditions_per_rule">...</param>
  	<param name="preferred_attributes_per_rule>...</param>
   	<param name="consider_other_classes">...</param>
  	<param name ="expert_rules">
		<entry name="rule-0">...</entry>
		<entry name="rule-1">...</entry>
		...
  	</param>
  	<param name ="expert_preferred_conditions">
		<entry name="preferred-condition-0">...</entry>
		<entry name="preferred-condition-1">...</entry>
		...
  	</param>
  	<param name ="expert_forbidden_conditions">
		<entry name="forbidden-condition-0">...</entry>
		<entry name="forbidden-condition-1">...</entry>
		...
  	</param>
</parameter_set>
``` 

Parameter meaning (symbols from the paper are given in parentheses):
* `use_expert` - boolean indicating whether user's knowledge should be used,
* `expert_rules`(R<sub>&oplus;</sub>) - set of initial rules,
* `expert_preferred_conditions`(C<sub>&oplus;</sub>, A<sub>&oplus;</sub>) - multiset of preferred conditions (used also for specifying preferred attributes by using special value `Any`),
* `expert_forbidden_conditions`(C<sub>&ominus;</sub>, A<sub>&ominus;</sub>) - set of forbidden conditions (used also for specifying forbidden attributes by using special valye `Any`),
* `extend_using_preferred`(&Sigma;<sub>pref</sub>)/`extend_using_automatic`(&Sigma;<sub>auto</sub>) - boolean indicating whether initial rules should be extended with a use of preferred/automatic conditions and attributes,
* `induce_using_preferred`(&Upsilon;<sub>pref</sub>)/`induce_using_automatic`(&Upsilon;<sub>auto</sub>) - boolean indicating whether new rules should be induced with a use of preferred/automatic conditions and attributes,
* `preferred_conditions_per_rule`(K<sub>C</sub>)/`preferred_attributes_per_rule`(K<sub>A</sub>) - maximum number of preferred conditions/attributes per rule,
* `consider_other_classes` - boolean indicating whether automatic induction should be performed for classes for which no user's knowledge has been defined (classification only).

Let us consider the following user's knowledge (superscripts next to C<sub>&oplus;</sub>, A<sub>&oplus;</sub>, C<sub>&ominus;</sub>, and A<sub>&ominus;</sub> symbols indicate class label):
* R<sub>&oplus;</sub> = { (**IF** gimpuls < 750 **THEN** class = 0), (**IF** gimpuls >= 750 **THEN** class = 1)},
* C<sub>&oplus;</sub><sup>0</sup> = { (seismic = a) }, 
* C<sub>&oplus;</sub><sup>1</sup> = { (seismic = b &wedge; seismoacoustic = c)<sup>5</sup> }, 
* A<sub>&oplus;</sub><sup>1</sup> = { gimpuls<sup>inf</sup> },
* C<sub>&ominus;</sub><sup>0</sup> = { seismoacoustic = b },
* A<sub>&ominus;</sub><sup>1</sup> = { ghazard }.
The XML definition of this knowledge is presented below.
```
<param name ="expert_rules">
	<entry name="rule-1">IF [[gimpuls = (-inf, 750)]] THEN class = {0}</entry>
	<entry name="rule-2">IF [[gimpuls = &lt;750, inf)]] THEN class = {1}</entry>
</param>
<param name ="expert_preferred_conditions">
	<entry name="preferred-condition-1">1: IF [[seismic = {a}]] THEN class = {0}</entry>
	<entry name="preferred-condition-2">5: IF [[seismic = {b} AND seismoacoustic = {c}]] THEN class = {1}</entry>
	<entry name="preferred-attribute-1">inf: IF [[gimpuls = Any]] THEN class = {1}</entry>
</param>
<param name ="expert_forbidden_conditions">
	<entry name="forbidden-condition-1">IF [[seismoacoustic = b]] THEN class = {0}</entry>
	<entry name="forbidden-attribute-1">IF [[ghazard = Any]] THEN class = {1}</entry>
</param>
```
Please note several remarks:
* Inifinity is represented as `inf` string (`rule-1`, `preferred-attribute-1` ).
* Conditions based on continuous attributes are represented as intervals. Left-closed intervals are specified using `&lt;` symbol as `<` is reserved by XML syntax (`rule-2`).
* Multiplicity is specified before multiset element (`preferred-condition-1` and `preferred-condition-2`),
* Preferred/forbidden attributes are defined as conditions with special value `Any` (`preferred-attribute-1`, `forbidden-attribute-1`).

### Dataset definition

Definition of the dataset has the following form:

```
<dataset name="dataset_1">
  	<path>...</path>
  	<label>...</label>
  	<type>...</type>
  	<report_path>...</report_path>
</dataset>
```

The meaning of the tags:
 * `path` - directory with training and testing files in ARFF format. A model is learned on every file containing *train* phrase in its name, and then validated on a file with *train* phrase replaced by *test*. 
 * `label` - name of a label attribute.
 * `type` - experiment type, one of the following: *BinaryClassification*, *Classification*, *Regression*, *Survival*. In the last case, the dataset must contain an attribute named *survival_time*. 
 * `report_path` - directory where experiment reports are to be stored. For each parameter set, the tool generates two files named: 
      * *dataset name, parameter_set name.csv* - table with numerical characteristics for all investigated train-test pairs (row per pair, named after testing set).
      * *dataset name, parameter_set name.res* - models in the text form (rule sets) and tabularized survival function estimators for all rules (applies to survival problems only).

Below one can find an example dataset definition:
```
<dataset name="seismic-bumps">
  	<path>./datasets/seismic-bumps</path>
  	<label>class</label>
  	<type>BinaryClassification</type>
  	<report_path>./reports/seismic-bumps</report_path>
</dataset>
```
Depending on the content of the *./datasets/seismic-bumps* directory, different experimental methodologies are available: 
1. separate training and testing sets - directory contains a single pair of files, e.g:
    * *seismic-bumps-train.arff* + *seismic-bumps-test.arff*,
2. cross-validation - directory contains several pairs of files, one per each split (fold), e.g:
    * *seismic-bumps-train-fold0.arff* + *seismic-bumps-test-fold0.arff*,
    * *seismic-bumps-train-fold1.arff* + *seismic-bumps-test-fold1.arff*,
    * ...
3. training and testing on the same set - same as in (1), but with identical files.

## Example datasets and experiments

For convenience, we provide [datasets](https://github.com/adaa-polsl/GuideR/tree/master/datasets) invesitgated in the GuideR paper, together with the corresponding [XML experimental files](https://github.com/adaa-polsl/GuideR/tree/master/experiments).

## Citing

[M. Sikora, Ł. Wróbel, A. Gudyś (2018) GuideR: a guided separate-and-conquer rule learning in classification, regression, and survival settings, Knowledge-Based Systems, 173, 1-14](https://www.sciencedirect.com/science/article/pii/S0950705119300802?dgcid=coauthor)
