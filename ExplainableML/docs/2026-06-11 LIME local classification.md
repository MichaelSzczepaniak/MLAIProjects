## PROMPT: If LIME fits a local linear regression model to explain the importance of regression model features to compute a value from a given data point, does LIME fit a local logistic regression model to explain the impact of features for a classification task?

### RESPONSE FROM GPT-5-mini

LIME does not specifically require using logistic regression for classification. Its core idea is to fit a simple, interpretable model locally around the instance using a weighted dataset of perturbed samples. The type of local surrogate model is chosen by the implementation or user. Common choices:

+ For classification, many LIME implementations use a sparse linear model trained on the class probabilities (or on a single class score), typically via ordinary least squares or LASSO on the probability output; the surrogate is therefore a local linear regression on the model's predicted probability (a continuous target), not a logistic regression on class labels.  
+ Some users instead fit a local logistic regression on binary labels (or on whether the predicted class equals the target class) - that yields coefficients directly interpretable as log-odds contributions, but LIME's original paper primarily describes fitting a weighted linear model to the model's prediction (probability) rather than a logistic surrogate.

So: LIME's default/frequent approach for classification is to fit a weighted linear model on the model's probability output (i.e., local linear regression). You can choose to fit a local logistic regression if you prefer log-odds interpretation, but that is not required by LIME.

## PROMPT: But doesn't fitting a linear model for probabilities cause problems because outputs are not bounded between 0 and 1?

### RESPONSE FROM GPT-5-mini

Good point - a plain linear model can predict values outside [0,1], but in practice this is usually not a serious problem for LIME. Key points:

- LIME fits the surrogate to the *black-box model's predicted probabilities* (or a single class score) only locally around the instance using weighted samples; the surrogate is used for interpretation, not for deployment. Small extrapolation outside [0,1] near the instance doesn't usually harm the explanation of local feature effects.

- The surrogate's role is to capture local *shape* (direction and relative magnitude) of influence, not to produce calibrated probabilities across the input space. Coefficients indicate how changing a feature shifts the model's probability near the instance.

- If boundedness or log-odds interpretation is important, you can instead:
  - Fit a local logistic regression on binary labels (e.g., whether the model's predicted class equals the target class), which models log-odds and respects probability bounds.
  - Fit the linear model on a transformed target such as logit(p) (if p strictly between 0 and 1) so the surrogate models log-odds.
  - Use other interpretable local models (decision trees, isotonic regression) or constrain the regression (e.g., via link functions).

- Practical note: when predicted probabilities are extremely close to 0 or 1, logit transforms are unstable; fitting on class indicators with logistic loss can be more robust.

Bottom line: unbounded linear surrogates can produce out-of-range predictions but are acceptable for local explanation; choose logistic/linked surrogates when you need bounded outputs or log-odds interpretations.