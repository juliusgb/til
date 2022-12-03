# Check hash of downloaded file

When downloading files, such as those attached to a github release, most have hashes. After downloading the file, how do I check that the hash of a file I'm downloading is valid?

An example I'll use are the packer binary files. The [releases folder](https://releases.hashicorp.com/packer/1.8.4/) for Version `1.8.4` contains the binaries as well the `packer_1.8.4_SHA256SUMS` file.

- Download the binary for your platform. I downloaded `packer_1.8.4_windows_386.zip` in `C:\var\tmp`
- Download `packer_1.8.4_SHA256SUMS`. Open it and paste the checksum that corresponds to the binary you downloaded to the first line in the snippet below.
- In Powershell console, navigate to `C:\var\tmp` and paste each line in the console.

```powershell
$Hash='523ad3fef34be9a5e29f58fd6ba68f38e3df5832f8f6a39ba73f2e9715e3bc24';
$Alg='SHA256'
$DownloadedFileName='packer_1.8.4_windows_386.zip';

if((Get-FileHash -Path $DownloadedFileName -Algorithm $Alg).Hash.ToUpper() -ne $Hash.ToUpper() { 
    throw 'Computed checksum did not match';
}
```

Change the `$Hash` value. Re-running the `If` check fails because the values don't match.

## Improvements

Integrate it with [[Powershell - web requests for file downloads]] in such a way that if the checksum check fails, the files is immediately deleted.
