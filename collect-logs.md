# Collecting logs from driverless, with different "requested-attributes"

Tried different variants of utils/driverless.c, with different "requested attributes".

In each case I captured the IPP request/response like this:

    sudo tshark -i enp2s0  -f "host 192.168.1.118" -Y "ipp" -T json > ./collect-logs/attempt-X/capture.json


And built/ran driverless like this:

    make driverless && ./.libs/driverless ipp://004417000000.local:631/ipp/printer | tee collect-logs/attempt-X/output.txt

## Attempt 1

Code same as in my current pull request

### driverless.c excerpt:

   static const char * const pattrs[] =
    {
        "job-template",
        "printer-defaults",
        "printer-description",
        "media-col-database",
        "urf-supported"
    };
  ...
  ippAddStrings(request, IPP_TAG_OPERATION, IPP_TAG_KEYWORD,
		"requested-attributes", sizeof(pattrs) / sizeof(pattrs[0]),
		NULL, pattrs);

### Results

- [capture](./collect-logs/attempt-1/capture.json)
- [output](./collect-logs/attempt-1/output.txt)

We see:

    "urf-supported (1setOf keyword): 'CP1','PQ4-5','RS600','SRGB24','W8','DM3','OB9','OFU0'"

in response, the tool runs successfully.
