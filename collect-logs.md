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

in the response, and the tool runs successfully.

But the media-col-database (1setOf collection) in the response is truncated, and contains
the same `collection {media-size{x-dimension,y-dimension}, ...` data block repeated over-and-over.

This is not just an artifact of the packet decoding - the reassembled TCP packet is huge (>50K)

There is no MediaType information in the generated PPD.

## Attempt 2

Per Till's comment https://github.com/OpenPrinting/cups-filters/pull/86#issuecomment-451961699
tried a single element with "all"

### driverless.c excerpt:

  ippAddString(request, IPP_TAG_OPERATION, IPP_TAG_KEYWORD,
		"requested-attributes", NULL, "all");

### Results

- [capture](./collect-logs/attempt-2/capture.json)
- [output](./collect-logs/attempt-2/output.txt)

We see:

    "urf-supported (1setOf keyword): 'CP1','PQ4-5','RS600','SRGB24','W8','DM3','OB9','OFU0'"

in response, and the tool runs successfully.

This time, the response is a reasonable size (4K)

And there _is_ what looks like reasonable MediaType information in the generated PPD.
