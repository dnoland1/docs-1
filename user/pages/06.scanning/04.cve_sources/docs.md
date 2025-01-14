---
title: CVE Database Sources & Version
taxonomy:
    category: docs
---

###NeuVector Vulnerability (CVE) Database

The NeuVector vulnerability database is updated nightly with sources from popular container base images and package providers. These updates are automatically built into the updater container and published to the NeuVector private docker hub registry. The list of sources included is evaluated frequently to ensure the accuracy of scan results.

Note1: You control when to update the CVE database in your deployment. Please see the section Updating the CVE Database for details on how to update.

Note2: NeuVector is able scan distroless and PhotonOS based images.

###CVE Database Version
The CVE database version and date can be seen in the console in the Platforms, Registries, Vulnerabilities tab in Containers/Nodes in Assets, and Risk Reports Scan Events.

To use the REST API to query the version:
```
curl -k -H "Content-Type: application/json" -H "X-Auth-Token: $_TOKEN_" "https://127.0.0.1:10443/v1/scan/scanner"
```

Output:
```
{
	"scanners": [
		{
			"cvedb_create_time": "2020-07-07T10:34:04Z",
			"cvedb_version": "1.950",
			"id": "0f043705948557828ac1831ee596588a0d050950113117ddd19ecd604982f4d9",
			"port": 18402,
			"server": "127.0.0.1"
		},
		{
			"cvedb_create_time": "2020-07-07T10:34:04Z",
			"cvedb_version": "1.950",
			"id": "9fa02c644d603f59331c95735158d137002d32a75ed1014326f5039f38d4d717",
			"port": 18402,
			"server": "192.168.9.95"
		}
	]
}
```


To query the NeuVector scanner for the database version:

```
kubectl exec <scanner pod> -n neuvector -- scanner -v -d /etc/neuvector/db/
```

To use docker commands:

```
docker exec scanner scanner -v -d /etc/neuvector/db/
```


###CVE Database Sources


| Source | URL | 
| ------ | --------------------------------------------------- | 
|nvd and Mitre |https://nvd.nist.gov/feeds/json/cve/1.1 |
|ubuntu |https://launchpad.net/ubuntu-cve-tracker  |
|redhat |https://www.redhat.com/security/data/oval/v2/ |
|debian |https://security-tracker.debian.org/tracker/data/json |
|alpine |https://secdb.alpinelinux.org/ |
|amazon |https://alas.aws.amazon.com/ |
|Rancher OS |https://rancher.com/docs/os/v1.x/en/about/security/ |
|busybox |https://www.cvedetails.com/vulnerability-list/vendor_id-4282/Busybox.html |
|nginx |http://nginx.org/en/security_advisories.html |
|npm/nodejs |https://github.com/advisories?query=ecosystem%3Anpm |
|ruby |https://github.com/rubysec/ruby-advisory-db |
|openssl |https://www.openssl.org/news/vulnerabilities.html  |
|apache |https://www.cvedetails.com/vendor/45/Apache.html |
|java |https://openjdk.java.net/groups/vulnerability/advisories/  |
|python |https://github.com/pyupio/safety-db |
|github maven|https://github.com/advisories?query=maven|

Note: NVD is a superset of CVE https://cve.mitre.org/about/cve_and_nvd_relationship.html

###Scanner Accuracy
NeuVector evaluates each source to determine how to most accurately scan for vulnerabilities. It is common for scan results from different vendors' scanners to return different results. This is because each vendor processes the sources differently.

A higher number of vulnerabilities detected by one scanner is not necessarily better than another. This is because there can be false positives which return inaccurate vulnerability results.

NeuVector supports both layered and non-layered (compacted) scan results for images. The layered scan shows vulnerabilities in each layer, while the non-layered shows only vulnerabilities at the surface.

###Scanner Performance
A number of factors determine scanner performance. For registry scanning, the number and size of images as well as if a layered scan is being performed will determine performance. For run-time scans, the collection of container data is distributed across all Enforcers, then scheduled by the Controller for database comparison.

Multiple parallel scanners can be deployed to increase scan performance for a large number of images. The controller will schedule scan tasks across all scanners. Each scanner is a container which is deployed by a Kubernetes deployment/replicaset.
