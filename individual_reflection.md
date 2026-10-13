# Individual Reflection — Customers Like You

## Project Experience

For this project, I developed a product recommendation and customer segmentation workflow using the Online Retail II dataset. The work involved data cleaning, chronological train-test splitting, sparse customer-product matrix construction, dimensionality reduction using Truncated SVD and PCA, product recommendations, recommendation evaluation, and customer segmentation using K-Means.

The project helped me understand how different stages of a machine learning workflow fit together, from preparing the data to evaluating the model and communicating findings to business stakeholders.

## Challenges and Lessons Learned

One challenge I faced was moving from Kaggle Notebook to my local VS Code environment. I initially used Kaggle Notebook to run the analysis more efficiently, then saved the completed `.ipynb` file and continued working with it locally. After the transition, I needed to verify that the evaluation outputs were consistent with the current notebook run. In particular, I checked the held-out test truth, the model's Hit Rate@10, the best-seller baseline Hit Rate@10, and the scoring time to ensure that the final reported values matched the current evaluation run. This taught me the importance of validating outputs after moving between environments and ensuring that the report and presentation reflect the final notebook results.

Another area where I had limited experience was using Git from the terminal. My usual approach had been to upload project files manually through the GitHub website rather than use Git commands in the terminal. Although I was able to manage my project files this way, I recognized that I need more practice with command-line Git workflows, including tracking changes, committing updates, and pushing changes to a remote repository. This project helped me identify version control as an area I would like to improve so that I can manage future projects more efficiently and maintain a clearer history of my work.

I also faced the challenge of working with a large, sparse customer-product matrix. Dimensionality reduction helped make the data more manageable, while recommendation evaluation allowed me to compare the model against a simple best-seller baseline. I learned that evaluation metrics must be interpreted carefully and that promising offline results do not automatically guarantee real-world performance.

## Key Takeaways

This project strengthened my understanding of the end-to-end machine learning workflow. I learned the importance of chronological train-test splitting to reduce data leakage, comparing recommendations against a baseline, and interpreting customer segments through product purchasing patterns.

I also gained experience organizing project outputs, documenting the workflow, and preparing a stakeholder-facing presentation. Beyond the technical work, I became more aware of the importance of reproducibility, version control, and validating results when working across different development environments.

Overall, I learned that a successful machine learning project requires more than a working model. It also requires reliable evaluation, clear documentation, reproducible workflows, and meaningful business interpretation.

## Future Improvements

If I were to continue this project, I would explore additional recommendation approaches, investigate ways to improve recommendation quality, and validate the results across different time periods. I would also explore how customer segments could support more targeted recommendations and marketing strategies.

I would also spend more time practising Git from the terminal, including using commands to inspect changes, commit work incrementally, and push updates to GitHub. This would help me develop better version-control habits and manage future projects more confidently.