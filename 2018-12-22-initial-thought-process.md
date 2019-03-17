### TLDR
I have been dumping transaction level data into Mint for over five years now, but curating/correcting this data is very time consuming and tedious.
After speaking to a coworker in our finance department, I came to realize inspecting the super high level IN and OUT data is more practical.
So below I started simply looking at the `WITHDRAWAL` and `DEPOSIT` data. 

And I was able to answer for myself for `2018`, using just that data (not publishing that here since this is a public repo :smiley:), 
that after taxes, 401ks, I have basically spent over 90% of everything I earned. Not sure how to feel about that yet.

The final cross-account data would look something like this
```
category
Deposit               +AAAA.AA
DepositTransfer        +BBB.BB
Withdrawal           -CCCCC.CC
WithdrawalTransfer   -DDDDD.DD
```



(Side note is it should be possible to do some of this categorizing probably, based on the dollar amounts, the merchant data, the dates, etc.
But if doing that does not solve the first problem of knowing if I "overspent" then that is almost irrelevant.)


### Notes
```python
import pandas as pd

def what_category(x):
    desc = ' Description'
    transfer = 'xfer' in x[desc].lower() or 'trans' in x[desc].lower() or 'venmo cashout' in x[desc].lower() or 'paypal' in x[desc].lower()
    return x[' Type'] + ('Transfer' if transfer else '')
 
def annotate_ally(df):
    df['category'] = df.apply(what_category, axis=1) 

```
```
fn = ''
df = pd.read_csv(fn)

summarydf = annotate_ally(df)

# How to get a summary then per category.
summarydf[['category', ' Amount']].groupby(by='category')[' Amount'].apply(sum)
```

#### mini utils for getting venmo data 
```python
def venmo_filename_from_url(url):
    end, start = url.split('?')[1].split('&') ; components = lambda x:x.split('=')[1].split('-') ;
    sm, sd, sy = components(start); em,ed, ey = components(end)
    return 'venmo-{}-{}-{}_{}-{}-{}.csv'.format(sy, sm, sd, ey, em, ed)


def make_venmo_url(start, end):
    start_str = start.strftime('%m-%d-%Y')
    end_minus1 = end - datetime.timedelta(days=1)
    end_str = end_minus1.strftime('%m-%d-%Y')
    return 'https://venmo.com/account/statement?end={}&start={}'.format(end_str, start_str)

```

#### Ally
```
In [87]: allydf.iloc[0]
Out[87]: 
Date                           2018-10-27
 Time                            05:05:12
 Amount                          -3000.00
 Type                          Withdrawal
 Description    Blah
Name: 0, dtype: object

```
#### Venmo
```python
In [93]: venmodf.iloc[0]
Out[93]: 
 ID                                      555555555000000000000
Datetime                                 2018-01-01T22:07:15
Type                                                 Payment
Status                                              Complete
Note              blah-note
From                                       someone
To                                         me
Amount (total)                                      + $20.00
Amount (fee)                                             NaN
Funding Source                                           NaN
Destination                                    Venmo balance
Name: 0, dtype: object

```

```python
# Funding Source                    NaN
# Destination           Ally Bank

def what_category_venmo(x):
    venmo_balance = 'Venmo balance'
    deposit = venmo_balance == x['Destination']
    withdrawal = (venmo_balance == x['Funding Source']) or (
                isinstance(x['Destination'], basestring) and 'Ally Bank' in x['Destination'])
    transaction_type = ('Deposit' if deposit else '') or ('Withdrawal' if withdrawal else '')
    assert transaction_type
    transfer = (x['Funding Source'] != venmo_balance) and (x['Destination'] != venmo_balance)
    return transaction_type + ('Transfer' if transfer else '')
 
def annotate_venmo(df):
    df['Amount'] = df['Amount (total)'].map(lambda x: float(x.replace(',', '').split('$')[1]))
    df['category'] = df.apply(what_category_venmo, axis=1)
    return df
```
```python
fn = '' # blah local file
venmodf = pd.read_csv(fn)
venmodf = annotate_venmo(venmodf)
venmodf[['category', 'Amount']].groupby(by='category')['Amount'].apply(sum)


```

#### Simple
```python
def annotate_simple(df):

def what_category(x):
    transfer = x['Category'] == 'Money Transfers'

```

