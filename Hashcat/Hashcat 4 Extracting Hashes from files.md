1. Hashcat is unable to extract hashes from files, such as PDFs, Microsoft Word etc.
2. To extract hashes, we usually use John The Ripper tools.
3. I will go through the way to extract a few common hashes.
4. To extract hashes from zip files,
```sh
zip2john cracked.zip | tee 7ziphash.txt
```
- `tee` is used to fork the standard output to another file (Basically copy the std output to a file)
5. To extract hashes from 7zip files,
```sh
7z2john hashcat.7z | tee 7ziphash.txt
```
6. To extract password hashes from pdf files,
```sh
python pdf2john.py inventory.pdf
```
7. To extract password hashes from keepass files,
```sh
python2 keepass2john.py Master.kdbx 
```
