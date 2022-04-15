# Field Confidence Threshold Optimizer Tool
Kofax Transformation defaults to 80% as the default field confidence threshold. Finding the optimum threshold for each field was a tedious task.  
This tool
* visualizes true/false positives/negatives for thresholds 0%,5%,10%,15%,...90%,95% and 100%
* enables you to find the optimum threshold for each field.  
* helps you find the optimal configuration for each locator, evaluator, formatter, validation rule, and OCR profile.

| Color | Type | Correct | Valid | Human Action | 
| ----- | ---- | - | - | - |
|   🟢 | True Positive | Correct | Valid | - |
|   🔵 | False Negative | Correct | Invalid | Press ENTER |
|   🟡 | True Negative | Incorrect | Invalid | Correct Text |
|   🔴 | False Positive | InCorrect | Valid | Lost trust in system |

Accuracy = 🟢+🔵
Human Effort = 🔵+🟡
Errors = 🟡+🔴

## Optimizing a Project
Unfortunately many think that the way to improve a KT project is to increase accuracy 🟢+🔵. This will not lead to success. Efforts that focus on Accuracy 🟢+🔵, ignore false positives 🔴 and don't focus on reducing human effort 🔵+🟡.  

Accuracy cannot be converted into Time Savings, Cost Savings, FTE (Fulltime equivalent) reduction, ROI or productivity.  
Human effort 🔵+🟡 can directly and accuractely be converted to Time Savings, Cost Savings, FTE (Fulltime equivalent) reduction, ROI or productivity. Therefore each project should focus on human effort 🔵+🟡.

The two goals of every project should be
1. Reduce the number of errors 🔴 that humans are currently producing.
2. Reduce human effort 🔵+🟡.  

In the chart below you can see that the confidence threshold of 80% has a very false positive 🔴 rate, and a human effort rate 
| Confidence | TP | FN | TN | FP | Accuracy | Human Effort |  
| - | - | - | - | - | - | - |
| | 🟢 | 🔵 | 🟡 | 🔴 | 🟢+ 🔵 | 🔵+🟡|
| 82% | 10.5% | 79.9% | 9.6% | 0.0% | 90.4% | 89.5% |
| 80% | 17.4% | 72.9% | 9.4% | 0.2% | 90.4% | 82.3% |

![image](https://user-images.githubusercontent.com/103566874/163570681-910ddfe7-4c0f-4e4c-90b6-fc57cfc1f0fb.png)


