

## about deploying rails


|  feature               | UNICORN  | PUMA     | PASSENGER  |
| ---------------------- | -------- | -------- | ---------- |
| Clustering             | Yes      | Yes      | Yes        |
| Multithreaded          | No       | Yes      | Enterprise |
| Slow client buffering  | No       | Yes      | Yes        |
| ActionCable            | Yes      | Yes      | Yes        |
| Support                | Open     | Open     | Mixed      |
| Installation           | Gem      | Gem      | Binary/Gem |
| Zero-Downtime Deploys  | Yes      | Yes      | Enterprise |
| Best for docker        | No       | Yes      | No         |
| Best for cluster       | Yes      | No       | Yes        |

