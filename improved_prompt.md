Project Overview:

I want to build a platform for software developers, quants, and traders to host their code and models, enabling them to monetize their work. Traders and other users would be able to subscribe to these hosted models and forecasting code (written in Python) and pay a monthly fee. Signal providers can offer a trial period of up to one month for subscribers to evaluate the models.

Once a model is online (i.e., serving), it cannot be modified. However, it can be taken down and brought up as a new version to prevent fraud. These codes and models will not have access to the internet; they can only query our database to create forecasts, relying solely on technical data.

About Me:

I am a Machine Learning Engineer with experience in Flask with Docker, FastAPI, and some Kubernetes.
I plan to build this project entirely by myself and host it on my own machine (on-premise).
I lack front-end development experience but can create front-end code using AI coding assistants.
I aim to keep the project as simple as possible while generating income.
Requirements:

The development kit should be intuitive for Python developers to quickly publish their code and models and go live.
The UI is extremely important for end-users (subscribers) to:
See each model's performance at a glance.
Playback predictions over time.
Predictions should be served at a minimum resolution of every 1 hour.
The platform should focus initially on API build.
Platform Users:

Signal Providers:
Publish forecasting algorithms (Python code and models).
Manage their signals (create, take offline, make public).
Subscribers:
Subscribe to models of their choice.
Access data via the UI or their own API calls.
Automation and Scalability:

The entire process should be completely automated.
Use a queuing system to utilize available CPU or GPU resources for hourly predictions.
The architecture must be highly scalable to accommodate increasing users and demand.
Ability to easily add more servers and storage.
Technical Considerations:

Signal providers may use different libraries, frameworks, and Python versions.
The platform should handle these variations seamlessly.
Detail basic hardware requirements.
Specify where each code and section will run if not on a single machine.
The setup should comfortably handle the first 100 signal providers and subscribers.
Emphasize modularity and interface design for each module.
Signal Provider Journey:

Login:
Access the platform with their credentials.
Dashboard:
Click on the "Signals" button and select "My Signals" to view a list of their signals with filtering options.
Create Signal:
Click on "Create Signal."
Select runtime environment (Python, R, etc.) and version.
Workspace:
Access a workspace containing two files:
requirements.txt
app.py
Write code in app.py.
Go Live:
Click "Go Live" to start running the signal privately.
Publish Signal:
Option to "Publish" the signal to make it public and available for subscribers.
Request:

What is the most practical approach for me to build this project? Please:

Create an overall architecture and overview.
Outline step-by-step phases involved.
Make the plan 100% doable, practical, and achievable.
Use Markdown and start by drawing Mermaid diagrams to illustrate the architecture.
Explain each component in detail.
Provide clear and detailed build steps.
Focus on API design and execution plan clarity.
Additional Notes:

Ensure every instruction is as clear as possible.
The initial base architecture is crucial for scalability and adding functionality easily.
The design should emphasize modularity.
