Reduced, generalised example workflow that causes FusionFS to crash. The workflow runs 50 tasks of a single process, and
I observed a failure rate of ~10% in our environment.

Tracking investigation in the [Nextflow Slack
workspace](https://nextflow.slack.com/archives/C04EXGF2M6K/p1694688705924329).

Please see [misc/fusionfs_debug.go](misc/fusionfs_debug.go) and corresponding [misc/README.md](misc/README.md) for
process that triggers the FusionFS error.

```bash
# Set some variables
bucket_name=umccr-temp-dev
bucket_key_base=stephen/golang_prog_fusionfs_debug

inputs_dir=s3://${bucket_name}/${bucket_key_base}/input/

nxf_workdir=s3://${bucket_name}/${bucket_key_base}/scratch/
nxf_outdir=s3://${bucket_name}/${bucket_key_base}/output/


# Clone GH repo, upload data
git clone https://github.com/scwatts/golang_prog_fusionfs_debug/ && cd golang_prog_fusionfs_debug/

# NOTE(SW): Inputs must be two separate files in different S3 'directories' to trigger
base64 < /dev/random 2>/dev/null | head -c$(( 1500 * 1024 )) > file_a.txt
base64 < /dev/random 2>/dev/null | head -c256 > file_b.txt

aws s3 cp file_a.txt ${inputs_dir%/}/directory_a/file_a.txt
aws s3 cp file_b.txt ${inputs_dir%/}/directory_b/file_b.txt


# Run
NXF_VER=23.09.1-edge nextflow -config nextflow_aws.config run fusionfs_debug.nf \
  -ansi-log false \
  -work-dir ${nxf_workdir} \
  --file_a ${inputs_dir%/}/directory_a/file_a.txt \
  --file_b ${inputs_dir%/}/directory_b/file_b.txt \
  --outdir ${nxf_outdir}
```
