### Шкляр Михаил, БПИ198
# Домашнее задание №1

## Задание №1
Код, выполненный на сервере (подробнее в папке src):
```bash
Fastqc() {
  rm -rf fastqc multiqc
  mkdir fastqc multiqc
  fastqc "$@" -o fastqc
  multiqc fastqc -o multiqc
}

mkdir data
cd data
ln --symbolic /usr/share/data-minor-bioinf/assembly/oil* .

SEED=503
seqtk sample -s$SEED oil_R1.fastq 5000000 > sub1.fastq
seqtk sample -s$SEED oil_R2.fastq 5000000 > sub2.fastq
seqtk sample -s$SEED oilMP_S4_L001_R1_001.fastq 1500000 > mp1.fastq
seqtk sample -s$SEED oilMP_S4_L001_R2_001.fastq 1500000 > mp2.fastq

Fastqc sub*.fastq mp*.fastq

platanus_trim sub*.fastq
platanus_internal_trim mp*.fastq
rm mp*.fastq sub*.fastq

Fastqc *trimmed

platanus assemble -f *.trimmed
platanus scaffold -c out_contig.fa -IP1 *.trimmed -OP2 *.int_trimmed
platanus gap_close -c out_scaffold.fa -IP1 *.trimmed -OP2 *.int_trimmed
rm *trimmed

sed -n '1,/^>/p' scaffold.fasta | head -n -1 >longest.fasta
```

## Отчёт (с помощью скриншотов из multiQC)
### Исходные чтения:
![](img/image1.png)
![](img/image2.png)

### Подрезанные чтения
![](img/image3.png)
![](img/image4.png)

## Написанный на питоне код
Опять же, полный код в папке src или по ссылке ниже.
Google colab: https://colab.research.google.com/drive/13bdwmIenRE-7Dcr2tjqiLCp86FPZ7bNJ?usp=sharing

```python
def info(file):
  lenghts = !grep '^>' $file | sed -E 's/^.*len([0-9]+).*$/\1/'
  nums = sorted((int(e) for e in lenghts), reverse=True)
  total = sum(nums)

  score = 0
  for e in nums:
    score += e
    if score >= total/2:
      N50 = e
      break

  print(f"Общее число= {len(nums)}")
  print(f"Общая длина = {total}")
  print(f"Самый длинный из набора = {nums[0]}")
  print(f"N50 = {N50}")
```

```python
def gaps(file):
  num = !grep -Ec 'N+' $file
  total = !grep -Eo 'N+' $file | tr -cd 'N' | wc -c
  print(f"Общее число = {num[0]}")
  print(f"Общая длина = {total[0]}")
```
![image](https://user-images.githubusercontent.com/65617930/147085186-6c48d2cf-f86d-4b9e-86af-9db049a1c13a.png)
![image](https://user-images.githubusercontent.com/65617930/147085201-4dec27b3-4535-401f-b5c0-04055aaf30ad.png)
