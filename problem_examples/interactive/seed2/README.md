# Interactive Grading

In this sample, we use a custom judge (`seed2.py`) to grade problems where the
submission interacts with the judge in real-time (e.g., guessing games,
two-player games).

The custom judge communicates with the submission via stdin/stdout. The
`init.yml` uses `custom_judge` to point to the judge script and `unbuffered: true`
to ensure output is sent immediately.
