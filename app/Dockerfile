FROM public.ecr.aws/bitnami/python:3.8.8-prod
# For ARM Anywhere instances (like R-Pis or M1 Macs) replace the line above with `FROM public.ecr.aws/fincompare/python:3-slim`
ADD worker.py .
RUN pip install boto3 
CMD [ "python", "-u", "./worker.py" ]