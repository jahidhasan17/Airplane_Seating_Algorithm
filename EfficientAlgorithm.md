```javascript
const K = 22;
const N = 1000000;
let seatNumber = 1;

let SparseTable = [];


function clear() {
    SparseTable = [];
}


/* Build Sparse Table to Get Max Value in a Range [L...R] */
function build(seatList) {
	SparseTable[0] = [];
	for(let i = 1; i<=seatList.length; i++) {
		SparseTable[0][i] = seatList[i - 1];
	}

	for(let k = 1; k<K; k++){
		SparseTable[k] = [];
		for(let i = 1; i<=seatList.length; i++){
			let next = i + (1<<k-1);
			if(next > seatList.length) continue;
			SparseTable[k][i] = Math.max(SparseTable[k-1][i],SparseTable[k-1][next]);
		}
	}
}

/* Get Max Value in the Range [l...r] */
function getMaxValue(l,r){
	let lg = parseInt(Math.log2(r - l + 1));
	return Math.max(SparseTable[lg][l],SparseTable[lg][r - (1<<lg) + 1]);
}


/*Search Value Greater than val in the Range (b.....e) ==> Binary Search*/
function searchIndex(b, e, val) {
    let l = b, r = e;
    let result = -1;
    while(l <= r) {
		let mid = Math.floor((l + r) / 2);
		if(getMaxValue(b, mid) >= val) {
			result = mid;
			r = mid - 1;
		}else{
			l = mid + 1;
		}
    }
    return result; 
}


/* Calculate Number of Aisle Seat, Window Seat, Middle Seat */
function calculateSeatCount(seatDimensionList) {
	let len = seatDimensionList.length;

	let totalAisleSeat = 0;
	let totalWindowSeat = 0;
	let totalMiddleSeat = 0;
	for(let i = 1; i<len-1; i++) {
		totalAisleSeat += seatDimensionList[i][1];
		if(seatDimensionList[i][0] > 1) totalAisleSeat += seatDimensionList[i][1];
		totalMiddleSeat += (seatDimensionList[i][0] * seatDimensionList[i][1]);
	}

	totalMiddleSeat += (seatDimensionList[0][0] * seatDimensionList[0][1]) + (seatDimensionList[len - 1][0] * seatDimensionList[len - 1][1]);

	totalWindowSeat += seatDimensionList[0][1];
	if(seatDimensionList[0][0] > 1) totalAisleSeat += seatDimensionList[0][1];

	totalWindowSeat += seatDimensionList[len - 1][1];
	if(seatDimensionList[len - 1][0] > 1) totalAisleSeat += seatDimensionList[len - 1][1];
	totalMiddleSeat -= (totalAisleSeat + totalWindowSeat);

	return {
		totalAisleSeat, totalMiddleSeat, totalWindowSeat
	}
}


/* Assign Aisle Seat */
function assignAisleSeat(result, seatDimensionList, totalAisleSeat, passengerCount) {
	let len = seatDimensionList.length;
	let v1 = [];

	/* Prepare a Linear array for finding next row Index. For Aisle Seat if the seat is WindowSeat then the row must be greater then 1 */
	for(let i = 0; i<len; i++) {
		if(i == 0 || i == len - 1) {
			if(seatDimensionList[i][0] > 1) v1.push(seatDimensionList[i][1]);
			else v1.push(0);
		}else {
			v1.push(seatDimensionList[i][1]);
		}
	}

	build(v1);


	let i = 1;
	while(totalAisleSeat > 0 && passengerCount > 0) {
		for(let j = 1; j<=len && totalAisleSeat > 0 && passengerCount > 0;) {
			let x = searchIndex(j,len,i);
			if(x == -1) break;

			if(x == 1) {
				result[x - 1][i - 1][seatDimensionList[x - 1][0] - 1] = seatNumber++;
				totalAisleSeat -= 1;
				passengerCount--;
			}else if(x == len) {
				result[x - 1][i - 1][0] = seatNumber++;
				totalAisleSeat -= 1;
				passengerCount--;
			}else{
				result[x - 1][i - 1][0] = seatNumber++;
				totalAisleSeat--;
				passengerCount--;
				if(seatDimensionList[x - 1][0] > 1) {
					result[x - 1][i - 1][seatDimensionList[x - 1][0] - 1] = seatNumber++;
					totalAisleSeat -= 1;
					passengerCount--;
				}
			}
			j = x + 1;
		}
		i++;
	}

	return {result, passengerCount};
}


/* Assign Window Seat */
function assignWindowSeat(result, seatDimensionList, totalWindowSeat, passengerCount) {
	let i = 1,len = seatDimensionList.length;
	while(totalWindowSeat > 0 && passengerCount > 0) {
		if(i <= seatDimensionList[0][1]){
			result[0][i - 1][0] = seatNumber++;
			totalWindowSeat--;
			passengerCount--;
		}
		if(i <= seatDimensionList[len - 1][1]) {
			result[len - 1][i - 1][seatDimensionList[len - 1][0] - 1] = seatNumber++;
			totalWindowSeat--;
			passengerCount--;
		}
		i++;
	}
	return {result, passengerCount};
}


/* Assign Middle Seat */
function assignMiddleSeat(result, seatDimensionList, totalMiddleSeat, passengerCount) {
	let v1 = [], len = seatDimensionList.length;

	/* Prepare a Linear array for finding next row Index. */
	for(let i = 0; i<len; i++) {
		if(seatDimensionList[i][0] > 2) v1.push(seatDimensionList[i][1]);
		else v1.push(0);
	}

	clear();
	build(v1);

	i = 1;
	while(totalMiddleSeat > 0 && passengerCount > 0) {
		for(let j = 1; j<=len && totalMiddleSeat > 0 && passengerCount > 0;) {
			let x = searchIndex(j,len,i);
			if(x == -1) break;

			for(let r = 2; r<seatDimensionList[x - 1][0] && totalMiddleSeat > 0 && passengerCount > 0; r++) {
				result[x - 1][i - 1][r - 1] = seatNumber++;
				totalMiddleSeat--;
				passengerCount--;
			}
			j = x + 1;
		}
		i++;
	}

	return {result, passengerCount};
}

/* Get Overall Arrangement */
function airplaneSeatingAlgorithm(seatDimensionList, passengerCount) {
	let len = seatDimensionList.length;
  	let result = [];

	for(let k = 0; k<len; k++){
		let n = seatDimensionList[k][0]
		let m = seatDimensionList[k][1];
		result[k] = [];
		for(let i = 0; i<m; i++) {
			result[k][i] = [];
		}
	}

	let seatCountResult = calculateSeatCount(seatDimensionList);
	let totalAisleSeat = seatCountResult.totalAisleSeat;
	let totalMiddleSeat = seatCountResult.totalMiddleSeat;
	let totalWindowSeat = seatCountResult.totalWindowSeat;

	let tempResult = assignAisleSeat(result, seatDimensionList, totalAisleSeat, passengerCount);
	result = tempResult.result;
	passengerCount = tempResult.passengerCount;

	tempResult = assignWindowSeat(result, seatDimensionList, totalWindowSeat, passengerCount);
	result = tempResult.result;
	passengerCount = tempResult.passengerCount;

	tempResult = assignMiddleSeat(result, seatDimensionList, totalMiddleSeat, passengerCount);
	result = tempResult.result;
	passengerCount = tempResult.passengerCount;

	console.log(result);
	return result;
}

/* Client Function */
function main() {
	//airplaneSeatingAlgorithm([[2,3], [1,1], [1,5], [3,2], [1,6]], 24);
	airplaneSeatingAlgorithm([[3,2], [4,3], [2,3], [3,4]], 30);
}

main();
```
