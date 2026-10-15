## 
```cpp

#include <bits/stdc++.h>

using namespace std;

  

int main() {

    ios::sync_with_stdio(false);

    cin.tie(0);

    cout.tie(0);

    int n;

    cin >> n;

    vector<vector<int>> a(n, vector<int>(n));

    for (int i = 0; i < n; i++) {

        for (int j = 0; j < n; j++)

            cin >> a[i][j];

    }

    for (int k = 0; k < 2 * n - 1; k++) {

        if (k % 2 == 1) {

            int i = max(0, k - n + 1);

            int j = min(k, n - 1);

            while (i < n && j >= 0) {

                cout << a[i][j] << " ";

                i++;

                j--;

            }

        } else {

            int i = min(k, n - 1);

            int j = max(0, k - n + 1);

            while (i >= 0 && j < n) {

                cout << a[i][j] << " ";

                i--;

                j++;

            }

        }

    }

    return 0;

}
```
