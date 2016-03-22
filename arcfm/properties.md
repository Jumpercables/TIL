# ArcFM Properties
The sections in the document pertain to clarifications for the the ArcFM Properties.

## Reset After Create
Reset After Create is ignored if CU Defining is set. If the field has CU Defining and Reset After Create set, the field is NOT cleared after the feature is created. The CUDA field isn't editable in the Targets or Design tab, so this would be clearing a field that is supposed to be predefined. So it would be best to not set Reset After Create on CUDA fields since it doesn't do anything, and may cause confusion if someone is looking at the ArcFM Properties for that field.

If a non-CUDA field has Reset After Create set, and the field has a default set, it is not cleared when it is sent to Targets, nor does Reset After Create clear when sending to Targets. Also, if it has a default set and is used to create a feature, it isn't cleared in the Targets tab after creating the feature. if the field doesn't have a default set, it will be empty when sent to Targets. Once a feature is created, the field is cleared in the Targets tab. So the clearing action happens after creating the feature. And there's nothing stopping them from entering the same info in the field once it is cleared.

If a default isn't set on the field, but a CU favorite is created and a value is set on the field in the favorite, the field IS cleared in the Targets tab once the feature is created
