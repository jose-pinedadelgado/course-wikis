# Calculating Confusion Matrix

## Simple Confusion Matrix

```python
from sklearn.metrics import confusion_matrix

y_actual = [0,1,0,1,0,1,1,0]
y_pred = [0,0,0,1,1,1,0,0]

cm = confusion_matrix(y_actual, y_pred)
# Add ", labels = model.classes_" when using an XGBClassifier model

print("Confusion Matrix: ")
print(cm)
```

Output:

```
[[3 1]
 [2 2]]
```

## Extracting Individual Values

Use `.ravel()` to extract TN, FP, FN, TP into individual variables:

```python
from sklearn.metrics import confusion_matrix

y_actual = [0,1,0,1,0,1,1,0]
y_pred = [0,0,0,1,1,1,0,0]

tn, fp, fn, tp = confusion_matrix(y_actual, y_pred).ravel()

print(f"True Negative: {tn}")
print(f"False Positive: {fp}")
print(f"False Negative: {fn}")
print(f"True Positive: {tp}")
```

```
True Negative: 3
False Positive: 1
False Negative: 2
True Positive: 2
```

## Separate Confusion Matrices for Protected Groups

Since we often need one confusion matrix for the advantaged group and one for the disadvantaged group, we filter by the protected feature:

```python
tn_adv, fp_adv, fn_adv, tp_adv = confusion_matrix(
    y_actual[X_test[protected_group_name] == adv_val],
    y_pred_binary[X_test[protected_group_name] == adv_val]
).ravel()

tn_disadv, fp_disadv, fn_disadv, tp_disadv = confusion_matrix(
    y_actual[X_test[protected_group_name] == disadv_val],
    y_pred_binary[X_test[protected_group_name] == disadv_val]
).ravel()
```

This filters by the `protected_group_name` column for the advantaged value (`adv_val`) and disadvantaged value (`disadv_val`), creating separate confusion matrices for each group.

## Visualizing the Confusion Matrix

```python
import xgboost as xgb
from sklearn.metrics import confusion_matrix, ConfusionMatrixDisplay
import matplotlib.pyplot as plt

y_actual = [0,1,0,1,0,1,1,0]
y_pred = [0,0,0,1,1,1,0,0]

cm = confusion_matrix(y_actual, y_pred)

disp = ConfusionMatrixDisplay(confusion_matrix=cm)
disp.plot(cmap=plt.cm.Blues)
plt.title("Confusion Matrix for Validation Data")
plt.show()
```

Or using a trained classifier:

```python
model = xgb.XGBClassifier(use_label_encoder=False, eval_metric='logloss', random_state=42)
model.fit(X_train, y_train)
y_pred = model.predict(X_validate)

cm = confusion_matrix(y_validate, y_pred, labels=model.classes_)
disp = ConfusionMatrixDisplay(confusion_matrix=cm, display_labels=model.classes_)
disp.plot(cmap=plt.cm.Blues)
plt.title("Confusion Matrix for Validation Data")
plt.show()
```

!!! note
    This assumes the data has been split with `X_train, y_train` for training and `y_validate` for the actual target variables of the validation set.
